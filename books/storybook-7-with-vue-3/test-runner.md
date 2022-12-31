---
title: ストーリーを自動テストする
---

これまでは `Storybook` を、ブラウザ上でコンポーネントを閲覧する用途で使用してきました。

本章では `Storybook` を用いた自動テストの手法を紹介します。

# パッケージのインストール

`Storybook` を用いた自動テストには [`@storybook/test-runner`](https://github.com/storybookjs/test-runner) を使用します。

```bash
$ yarn add -D @storybook/test-runner@0.10.0-next.3
```

`test-runner` は、起動している `Storybook` に対して各ストーリーを [Playwright](https://playwright.dev/) を用いて開きます。

ストーリーを開いた際にエラーが発生した場合はテストを失敗とみなすため、コンポーネント及びストーリーが壊れていないかを自動で担保できるようになります。

また、従来の `Jest` などを用いたコンポーネントのユニットテストの場合、 `Node.js` 及び `JSDOM` で実行するシミュレーションに限られました。これを `Storybook` を用いたテストにすることで、 `Chrome` などのクロスブラウザで、実際と同様の環境でのテストができるようになりました。

# テストの実行

`test-runner` を実行する前に、 `Storybook` を起動し、その URL を指定する必要があります。ここでは `6006` 番ポートで `Storybook` を起動するように指定しておきます。

```bash
$ yarn storybook dev --port 6006
```

`6006` 番ポートで `Storybook` が起動したので、そこに対して `test-runner` を実行します。

```bash
$ yarn test-storybook --url http://localhost:6006
```

これまで作成したすべてのストーリーが `chromium` ブラウザで開けたのでテスト成功です。

```bash
 PASS   browser: chromium  src/stories/MyButton.stories.ts
 PASS   browser: chromium  src/stories/MyHeader.stories.ts
 PASS   browser: chromium  src/stories/MyPage.stories.ts
 PASS   browser: chromium  src/stories/SignUpForm.stories.ts

Test Suites: 4 passed, 4 total
Tests:       8 passed, 8 total
Snapshots:   0 total
Time:        3.768 s
Ran all test suites.
✨  Done in 6.54s.
```

# ユーザーインタラクションをテストする

`test-runner` を実行するだけで、すべてのストーリーがエラー無く開けることは確認できました。

しかし、コンポーネントが意図通りの挙動をしていることを担保するために、ユーザーインタラクションを組み合わせることができます。

おさらいになりますが、 `SignUpForm` コンポーネントのストーリーでは、以下のようなユーザーインタラクションが定義されていました。

```ts:src/stories/SignUpForm.stories.ts
// 前略

export const Complete: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const nameInput = canvas.getByLabelText("Name", { selector: "input" });
    const ageInput = canvas.getByLabelText("Age", { selector: "input" });
    const submitButton = canvas.getByRole("button", { name: "確定" });

    await userEvent.type(nameInput, "sasaki");
    await userEvent.clear(ageInput);
    await userEvent.type(ageInput, "30");
    await userEvent.click(submitButton);
  },
};

export const Error: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const nameInput = canvas.getByLabelText("Name", { selector: "input" });
    const ageInput = canvas.getByLabelText("Age", { selector: "input" });
    const submitButton = canvas.getByRole("button", { name: "確定" });

    await userEvent.type(nameInput, "");
    await userEvent.clear(ageInput);
    await userEvent.type(ageInput, "17");
    await userEvent.click(submitButton);
  },
};

export default meta
```

上記のユーザーインタラクションは、フォーム入力とボタン押下までを行っていますが、その結果は目視することが前提で機械的に正しく動いているかの判定はしていませんでした。

フォームの入力内容に応じて、エラーメッセージが出ること、出ないことを `test-runner` で判別できるようにします。

テストコードでのアサーションには `Jest` を用いることが多く、 `Storybook` からもパッケージが配信されているのでそれを使います。

```bash
$ yarn add -D @storybook/jest
```

`SignUpForm` のストーリーを以下のように修正します。

```ts:src/stories/SignUpForm.stories.ts
import { expect } from "@storybook/jest";

// 中略

export const Complete: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const nameInput = canvas.getByLabelText("Name", { selector: "input" });
    const ageInput = canvas.getByLabelText("Age", { selector: "input" });
    const submitButton = canvas.getByRole("button", { name: "確定" });

    await userEvent.type(nameInput, "sasaki");
    await userEvent.clear(ageInput);
    await userEvent.type(ageInput, "30");
    await userEvent.click(submitButton);

    // エラーメッセージが表示されていないことをアサート
    expect(await canvas.queryByText("名前を入力してください")).toBeNull();
    expect(await canvas.queryByText("18歳以上でなければ登録できません")).toBeNull();
  },
};

export const Error: Story = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);

    const nameInput = canvas.getByLabelText("Name", { selector: "input" });
    const ageInput = canvas.getByLabelText("Age", { selector: "input" });
    const submitButton = canvas.getByRole("button", { name: "確定" });

    await userEvent.type(nameInput, "");
    await userEvent.clear(ageInput);
    await userEvent.type(ageInput, "17");
    await userEvent.click(submitButton);

    // エラーメッセージが表示されていることをアサート
    expect(await canvas.queryByText("名前を入力してください")).toBeTruthy();
    expect(await canvas.queryByText("18歳以上でなければ登録できません")).toBeTruthy();
  },
};

export default meta
```

これでユーザーインタラクション後のコンポーネントの振る舞いをテストできるようになりました。

試しにエラーメッセージを出すロジックにバグを仕込んでみましょう。

```ts:src/components/SignUpForm.vue
function onSubmit() {
  // エラーを出す条件を逆にしてしまった
  errors.name = state.name !== "" ? "名前を入力してください" : "";
  errors.age = state.age > 18 ? "18歳以上でなければ登録できません" : "";

  if (errors.name === "" && errors.age === "") {
    // TODO: ここでログイン処理をしてトップページに戻る
  }
}
```

テストを実行すると、以下のようなエラーが出るようになりました。 (一部抜粋)

```bash
 PASS   browser: chromium  src/stories/MyButton.stories.ts
 PASS   browser: chromium  src/stories/MyHeader.stories.ts
 PASS   browser: chromium  src/stories/MyPage.stories.ts
 FAIL   browser: chromium  src/stories/SignUpForm.stories.ts
  ● SignUpForm › Complete › play-test

    page.evaluate: StorybookTestRunnerError:
    An error occurred in the following story. Access the link for full output:
    http://localhost:6006/?path=/story/signupform--complete&addonPanel=storybook/interactions/panel

    Message:
     expect(received).toBeNull()

    Received: <span class="error" data-v-0a57caa1="">名前を入力してください</span>


    --------------------------------------------------

    Browser logs:

    error: {"matcherResult":{"message":"expect(received).toBeNull()\n\nReceived: <span class=\"error\" data-v-0a57caa1=\"\">名前を入力してください</span>","pass":false}}
```

# 関連リンク

https://storybook.js.org/docs/7.0/vue/writing-tests/test-runner

https://storybook.js.org/docs/7.0/vue/writing-tests/introduction