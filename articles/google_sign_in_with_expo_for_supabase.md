---
title: "Expo + SupabaseでGoogle Sign inのエラーに詰まった時の対応方法"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["supabase", "google", "expo", "reactnative"]
published: false
---

# はじめに

Supabaseの方であらかじめ、https://www.youtube.com/watch?v=vojHmGUGUGc や https://supabase.com/docs/guides/auth/social-login/auth-google の方に公式の記事・動画を出してくれています。そちらに則っていれば基本大丈夫だと思います。
vibe codingから始めたという背景もあって、私自身この問題にかなり時間を使ってしまいました。
僕と同じような状況になった方向け、かつ自分の備忘録としてこちら記載することにしました。

# 最初のアプローチ (expo-auth-session/providers/google)

最初は[bolt.new](https://bolt.new/?rid=fn114z)というAI coding agentを触ってみて、
- 「おお、結構コーディングできるやん」
- 「しかもデザインもよしなに実装してくれる」
- 「これならフロント得意じゃない俺でもアプリ開発できるかも」
と思い、今のプロジェクトを始めたのがきっかけです。
本業が終わってから実装するということもあり、かなりの部分をLLMに頼りながら実装していました。

ライブラリの選定理由などはプロンプトで指定してましたが、使用するライブラリなどはかなりLLMに任せていました。
ということで最初は`expo-auth-session/providers/google`を使っていました。
コードは以下のような感じでした。

```typescript
import * as WebBrowser from 'expo-web-browser';
import * as Google from 'expo-auth-session/providers/google';
import { AuthSessionResult } from 'expo-auth-session';

WebBrowser.maybeCompleteAuthSession();

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [session, setSession] = useState<Session | null>(null);
  const [loading, setLoading] = useState(true);
  const [initialized, setInitialized] = useState(false);

  // Configure Google Auth
  const [request, response, promptAsync] = Google.useAuthRequest({
    androidClientId: process.env.EXPO_PUBLIC_GOOGLE_ANDROID_CLIENT_ID,
    iosClientId: process.env.EXPO_PUBLIC_GOOGLE_IOS_CLIENT_ID,
    webClientId: process.env.EXPO_PUBLIC_GOOGLE_WEB_CLIENT_ID,
    expoClientId: process.env.EXPO_PUBLIC_GOOGLE_CLIENT_ID,
    responseType: "id_token",
    scopes: ['profile', 'email'],
    redirectUri: makeRedirectUri({
      scheme: Constants.expoConfig?.scheme || 'myapp'}),
  });

const handleGoogleToken = async (token: string | undefined) => {
    if (!token) return;

    try {
      const { error } = await supabase.auth.signInWithIdToken({
        provider: 'google',
        token,
      });

      if (error) throw error;
    } catch (error) {
      console.error('Error signing in with Google:', error);
      throw error;
    }
  };

  const signInWithGoogle = async () => {
    try {
      if (!promptAsync) return;
      const result = await promptAsync();
      if (result.type !== 'success') {
        console.log("Google sign in was cancelled or failed");
        throw new Error('Google sign in was cancelled or failed');
      }
    } catch (error) {
      console.error('Error initiating Google sign in:', error);
      throw error;
    }
  };

~~割愛~~

}
```

## OAuth clientの作成
Google CloudからOAuthのClient Credentialsを以下の流れで作成しました。[（Supabase公式記事参照）](https://supabase.com/docs/guides/auth/social-login/auth-google?queryGroups=platform&platform=react-native)

> 1. コンソールの Credentials ページで、Google Cloud プロジェクトの OAuth 認証情報を設定します。新しい OAuth クライアント ID を作成するときは、アプリをビルドするモバイル オペレーティング システムに応じて Android または iOS を選択します。
> - Android の場合、画面の指示に従って、Android アプリの署名に使用する SHA-1 証明書フィンガープリントを指定します。
>   - SHA-1証明書のフィンガープリントは、ローカルでのテスト用と本番用で異なるセットになります。Google Cloud Consoleに両方を追加し、SupabaseダッシュボードにすべてのクライアントIDを追加してください。
> - iOSの場合、画面の指示に従ってアプリのバンドルID、アプリがApple App Storeで公開済みの場合はApp Store IDとチームIDを指定します。
> 2. OAuth同意画面を設定します。この情報は、アプリに同意を与える際にユーザーに表示されます。特に、アプリのプライバシーポリシーと利用規約へのリンクが設定されていることを確認してください。
> 3. 最後に、Supabase DashboardのGoogleプロバイダの「Client IDs」に、ステップ1のクライアントIDを追加します。

# 二つ目のアプローチ (@react-native-google-signin/google-signin)


