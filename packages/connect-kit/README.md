# `@farcaster/connect-kit`

A React library that makes it easy to add Sign In With Farcaster to your application.

## Getting Started

### Installation

Install ConnectKit and its peer dependency [viem](https://viem.sh/).

```sh
npm install @farcaster/connect-kit viem
```

**Note:** ConnectKit is a [React](https://reactjs.org/) library. If you're using another frontend framework, take a look at the [`@farcaster/connect`](../connect/) client library instead.

### Import

Import styles and functions.

```tsx
import "@farcaster/connect-kit/styles.css";
import { ConnectKitProvider } from "@farcaster/connect-kit";
```

### Add Provider

Wrap your application with `ConnectKitProvider`.

```tsx
const config = {
  // For a production app, replace this with an Optimism Mainnet
  // RPC URL from a provider like Alchemy or Infura.
  domain: "example.com",
  siweUri: "https://example.com/login",
  rpcUrl: "https://mainnet.optimism.io",
  relay: "https://relay.farcaster.xyz",
};

const App = () => {
  return (
    <ConnectKitProvider config={config}>{/* Your App */}</ConnectKitProvider>
  );
};
```

### Troubleshooting

Projects using [Create React App (CRA)](https://create-react-app.dev/) may run into TypeScript version conflicts, as `react-scripts@5.0.1` expects a peer dependency of TypeScript version `^3.2.1 || ^4`, while both viem and ConnectKit require `>=5.0.4`.

To resolve this issue:
- Install the latest version of TypeScript: `npm i typescript -D`
- Add an override for `react-scripts` to your package.json file, to remove the version ceiling:
```json
"overrides": {
  "react-scripts": {
    "typescript": ">3.2.1"
  }
}
```

**Note**: Always be careful with overrides and ensure they are compatible with the packages using them. 

## Add the connect button

Then, in your app, import and render the `ConnectButton` component.

```tsx
import { ConnectButton } from "@farcaster/connect-kit";

export const Login = ({ nonce }: { nonce: string }) => {
  return <ConnectButton nonce={nonce} />;
};
```

A Sign in with Farcaster button will be rendered. When the user clicks it they will be prompted to complete sign in using their Farcaster wallet application. Once they complete sign in a signed message will be returned to your callback.

## Examples

You can find official examples [here](https://github.com/farcasterxyz/connect-monorepo/tree/main/examples).

## Components

### `ConnectKitProvider`

Wrap your application in a `ConnectKitProvider` to use Farcaster Connect.

```tsx
const config = {
  domain: "example.com",
  siweUri: "https://example.com/login",
  rpcUrl: "https://mainnet.optimism.io",
  relay: "https://relay.farcaster.xyz",
};

const App = () => {
  return (
    <ConnectKitProvider config={config}>{/* Your App */}</ConnectKitProvider>
  );
};
```

#### Props

| Prop     | Type               | Required | Description                                           |
| -------- | ------------------ | -------- | ----------------------------------------------------- |
| `config` | `ConnectKitConfig` | Yes      | Configuration object. See options in the table below. |

`config` options:

| Parameter | Type     | Required | Description                         | Default                       |
| --------- | -------- | -------- | ----------------------------------- | ----------------------------- |
| `domain`  | `string` | Yes      | Domain of your application.         | None                          |
| `siweUri` | `string` | Yes      | A URI identifying your application. | None                          |
| `relay`   | `string` | No       | Farcaster Connect relay server URL  | `https://relay.farcaster.xyz` |
| `rpcUrl`  | `string` | No       | Optimism RPC server URL             | `https://mainnet.optimism.io` |
| `version` | `string` | No       | Farcaster Connect version           | `v1`                          |

### `ConnectButton`

The main component. Renders a "Sign in With Farcaster" button that prompts the user to scan a QR code with their phone on web or redirects directly on a mobile device. You can use a callback prop or a hook to access the user's authentication status and profile information.

```tsx
import { ConnectButton } from "@farcaster/connect-kit";

export const Login = ({ nonce }: { nonce: string }) => {
  return (
    <ConnectButton
      nonce={nonce}
      onSuccess={({ fid, username }) =>
        console.log(`Hello, ${username}! Your fid is ${fid}.`)
      }
    />
  );
};
```

#### Props

| Prop               | Type                                | Required | Description                                                                         | Default              |
| ------------------ | ----------------------------------- | -------- | ----------------------------------------------------------------------------------- | -------------------- |
| `timeout`          | `number`                            | No       | Relay server timeout in ms.                                                         | `300000` (5 minutes) |
| `interval`         | `number`                            | No       | Relay server polling interval in ms.                                                | `1500` (1.5 seconds) |
| `nonce`            | `string`                            | No       | Random nonce to include in the Sign In With Farcaster message.                      | None                 |
| `notBefore`        | `string`                            | No       | Time when the SIWF message becomes valid. ISO 8601 datetime string.                 | None                 |
| `expirationTime`   | `string`                            | No       | Time when the SIWF message expires. ISO 8601 datetime string.                       | None                 |
| `requestId`        | `string`                            | No       | An optional system specific ID to include in the SIWF message.                      | None                 |
| `onSuccess`        | `(res: UseSignInData) => void`      | No       | Callback invoked when sign in is complete and the user is authenticated.            | None                 |
| `onStatusResponse` | `(res: UseWatchStatusData) => void` | No       | Callback invoked when the component receives a status update from the relay server. | None                 |
| `onError`          | `(error: ConnectError) => void`     | No       | Error callback function.                                                            | None                 |
| `debug`            | `boolean`                           | No       | Render a debug panel displaying internal ConnectKit state.                          | `false`              |

## Hooks

### `useSignIn`

Hook for signing in a user. Connects to the relay server, generates a QR code and sign in link to present to the user, and polls relay server for wallet signature.

```tsx
import { useAppClient } from "@farcaster/connect-kit";

function App() {
  const {
    signIn,
    qrCodeUri,
    data: { username },
    onSuccess: ({ fid }) => console.log("Your fid:", fid);
  } = useSignIn();

  return (
    <div>
      <button onClick={signIn}>Sign In</button>
      {qrCodeUri && (
        <span>
          Scan this: <img src={qrCodeUri} />
        </span>
      )}
      {username && `Hello, ${username}!`}
    </div>
  );
}
```

#### Parameters

| Parameter          | Type                                | Required | Description                                                                         | Default              |
| ------------------ | ----------------------------------- | -------- | ----------------------------------------------------------------------------------- | -------------------- |
| `timeout`          | `number`                            | No       | Relay server timeout in ms.                                                         | `300000` (5 minutes) |
| `interval`         | `number`                            | No       | Relay server polling interval in ms.                                                | `1500` (1.5 seconds) |
| `nonce`            | `string`                            | No       | Random nonce to include in the Sign In With Farcaster message.                      | None                 |
| `notBefore`        | `string`                            | No       | Time when the SIWF message becomes valid. ISO 8601 datetime string.                 | None                 |
| `expirationTime`   | `string`                            | No       | Time when the SIWF message expires. ISO 8601 datetime string.                       | None                 |
| `requestId`        | `string`                            | No       | An optional system specific ID to include in the SIWF message.                      | None                 |
| `onSuccess`        | `(res: UseSignInData) => void`      | No       | Callback invoked when sign in is complete and the user is authenticated.            | None                 |
| `onStatusResponse` | `(res: UseWatchStatusData) => void` | No       | Callback invoked when the component receives a status update from the relay server. | None                 |
| `onError`          | `(error: ConnectError) => void`     | No       | Error callback function.                                                            | None                 |

#### Returns

```ts
  {
    signIn: () => void;
    reconnect: () => void;
    isSuccess: boolean;
    isPolling: boolean;
    isError: boolean;
    error: ConnectError;
    channelToken: string;
    connectUri: string;
    qrCodeUri: string;
    data: {
        state: "pending" | "complete";
        nonce: string;
        message: string;
        signature: string;
        fid: number;
        username: string;
        bio: string;
        displayName: string;
        pfpUrl: string;
    },
    validSignature: boolean;
  };
```

| Parameter          | Description                                                                                      |
| ------------------ | ------------------------------------------------------------------------------------------------ |
| `signIn`           | Call this function to connect to the relay and poll for a signature.                             |
| `reconnect`        | Reconnect to the relay and try again. Use in the event of an error.                              |
| `isSuccess`        | True when the relay server returns a valid signature.                                            |
| `isPolling`        | True when the relay state is `"pending"` and the app is polling the relay server for a response. |
| `isError`          | True when an error has occurred.                                                                 |
| `error`            | `ConnectError` instance.                                                                         |
| `channelToken`     | Connect relay channel token.                                                                     |
| `connectUri`       | Sign in With Farcaster URL to present to the user. Links to Warpcast in v1.                      |
| `qrcodeUri`        | QR code image data URL encoding `connectUri`. Links to Warpcast in v1.                           |
| `data.state`       | Status of the sign in request, either `"pending"` or `"complete"`                                |
| `data.nonce`       | Random nonce used in the SIWE message. If you do not provide a custom nonce, read this value.    |
| `data.message`     | The generated SIWE message.                                                                      |
| `data.signature`   | Hex signature produced by the user's Warpcast wallet.                                            |
| `data.fid`         | User's Farcaster ID.                                                                             |
| `data.username`    | User's Farcaster username.                                                                       |
| `data.bio`         | User's Farcaster bio.                                                                            |
| `data.displayName` | User's Farcaster display name.                                                                   |
| `data.pfpUrl`      | User's Farcaster profile picture URL.                                                            |
| `validSignature`   | True when the signature returned by the relay server is valid.                                   |

### `useUserData`

Hook for reading information about the authenticated user.

```tsx
import { useUserData } from "@farcaster/connect-kit";

function App() {
  const {
    isAuthenticated,
    userData: { fid, pfpUrl, username, displayName, bio },
  } = useUserData();

  return (
    <div>
     { isAuthenticated ? <p>Hello, {username}! Your fid is {fid}.<p>  : <p>You're not signed in.</p>}
    </div>
  );
}
```

#### Returns

```ts
  {
    isAuthenticated: boolean;
    userData: {
        fid: number;
        username: string;
        bio: string;
        displayName: string;
        pfpUrl: string;
    },
  };
```

| Parameter              | Description                           |
| ---------------------- | ------------------------------------- |
| `isAuthenticated`      | True if the user is authenticated.    |
| `userData.fid`         | User's Farcaster ID.                  |
| `userData.username`    | User's Farcaster username.            |
| `userData.bio`         | User's Farcaster bio.                 |
| `userData.displayName` | User's Farcaster display name.        |
| `userData.pfpUrl`      | User's Farcaster profile picture URL. |

### `useSignInMessage`

Hook for reading the SIWE message and signature used to authenticate the user.

```tsx
import { useUserData } from "@farcaster/connect-kit";

function App() {
  const { message, signature } = useSignInMessage();

  return (
    <div>
      <p>You signed: {message}</p>
      <p>Your signature: {signature}</p>
    </div>
  );
}
```

#### Returns

```ts
{
  message: string;
  signature: string;
}
```

| Parameter   | Description                                       |
| ----------- | ------------------------------------------------- |
| `message`   | SIWE message signed by the user.                  |
| `signature` | Signature produced by the user's Warpcast wallet. |
