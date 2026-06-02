# BullMint_SOLANA_NFT_DEVnet
Start to finish Devnet React app that connects a Solana wallet and mints a standard Solana NFT using Metaplex Token Metadata through Umi.


# STEP 1 | Create the app

npm create vite@latest solana-nft-minter -- --template react-ts
cd solana-nft-minter

Install dependencies: 

npm install \
  @solana/web3.js@1 \
  @solana/wallet-adapter-base \
  @solana/wallet-adapter-react \
  @solana/wallet-adapter-react-ui \
  @solana/wallet-adapter-wallets \
  @metaplex-foundation/umi \
  @metaplex-foundation/umi-bundle-defaults \
  @metaplex-foundation/umi-signer-wallet-adapters \
  @metaplex-foundation/mpl-token-metadata


# STEP 2 |  Your NFT metadata file

You need a public metadata JSON URL. Upload this JSON somewhere public: Arweave, IPFS, Pinata, NFT.Storage, your own server, etc.

{
  "name": "THE GREATEST SOLANA NFT EVER",
  "symbol": "BULLBREW",
  "description": "A test NFT minted on Solana Devnet using Wallet Adapter and Metaplex Umi.",
  "image": "https://your-public-url.com/image.png",
  "attributes": [
    {
      "trait_type": "Network",
      "value": "Devnet"
    },
    {
      "trait_type": "Standard",
      "value": "Token Metadata"
    }
  ],
  "properties": {
    "files": [
      {
        "uri": "https://your-public-url.com/image.png",
        "type": "image/png"
      }
    ],
    "category": "image"
  }
}




Note: The uri you pass into the minting code should point to this JSON file, not directly to the image.


# STEP 3 | File Strcture

Create this structure:

solana-nft-minter/
  src/
    App.tsx
    main.tsx
    SolanaProvider.tsx
    MintNft.tsx
    index.css
  package.json
  index.html
  vite.config.ts

  
# STEP 4 | src/SolanaProvider.tsx

import { ReactNode, useMemo } from "react";
import {
  ConnectionProvider,
  WalletProvider,
} from "@solana/wallet-adapter-react";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";
import { PhantomWalletAdapter } from "@solana/wallet-adapter-wallets";
import { WalletModalProvider } from "@solana/wallet-adapter-react-ui";
import { clusterApiUrl } from "@solana/web3.js";

import "@solana/wallet-adapter-react-ui/styles.css";

export function SolanaProvider({ children }: { children: ReactNode }) {
  const network = WalletAdapterNetwork.Devnet;
  const endpoint = useMemo(() => clusterApiUrl(network), [network]);

  const wallets = useMemo(() => [new PhantomWalletAdapter()], []);

  return (
    <ConnectionProvider endpoint={endpoint}>
      <WalletProvider wallets={wallets} autoConnect>
        <WalletModalProvider>{children}</WalletModalProvider>
      </WalletProvider>
    </ConnectionProvider>
  );
}

# Step 5 | src/MintNft.tsx


import { useMemo, useState } from "react";
import { useWallet } from "@solana/wallet-adapter-react";
import { WalletMultiButton } from "@solana/wallet-adapter-react-ui";

import { createUmi } from "@metaplex-foundation/umi-bundle-defaults";
import { generateSigner, percentAmount } from "@metaplex-foundation/umi";
import {
  createNft,
  mplTokenMetadata,
} from "@metaplex-foundation/mpl-token-metadata";
import { walletAdapterIdentity } from "@metaplex-foundation/umi-signer-wallet-adapters";

const RPC_ENDPOINT = "https://api.devnet.solana.com";

// MUST be a real public JSON metadata URL.
const NFT_METADATA_URI = "https://your-public-url.com/metadata.json";

export function MintNft() {
  const wallet = useWallet();

  const [loading, setLoading] = useState(false);
  const [mintAddress, setMintAddress] = useState("");
  const [error, setError] = useState("");

  const umi = useMemo(() => {
    return createUmi(RPC_ENDPOINT)
      .use(walletAdapterIdentity(wallet))
      .use(mplTokenMetadata());
  }, [wallet]);

 async function mintNft() {
  setError("");
  setMintAddress("");

  if (!wallet.connected || !wallet.publicKey) {
    setError("Connect your wallet first.");
    return;
  }

  if (NFT_METADATA_URI.includes("your-public-url.com")) {
    setError("Replace NFT_METADATA_URI with a real hosted metadata JSON URL.");
    return;
  }

try {
  setLoading(true);

  const mint = generateSigner(umi);

  await createNft(umi, {
    mint,
    name: "BEST SOL NFT EVER",
    symbol: "THEBULLBREW",
    uri: NFT_METADATA_URI,
    sellerFeeBasisPoints: percentAmount(5),
  }).sendAndConfirm(umi);

  setMintAddress(mint.publicKey.toString());
} catch (err) {
  console.error("Mint error:", err);
  setError(err instanceof Error ? err.message : "Mint failed.");
} finally {
  setLoading(false);
}



# STEP 6 | Create src/App.tsx


Replace everything inside src/App.tsx with this:


import { MintNft } from "./MintNft";

export default function App() {
  return (
    <main className="page">
      <MintNft />
    </main>
  );
}


This tells the app to display your NFT minting component.


# STEP 7 | Update src/main.tsx

Replace everything inside src/main.tsx with this:

import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import { SolanaProvider } from "./SolanaProvider";
import "./index.css";

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <SolanaProvider>
      <App />
    </SolanaProvider>
  </React.StrictMode>
);

This wraps your whole app with the Solana wallet connection provider.


# STEP 8 | Update src/index.css

Replace your src/index.css with this simple version:

* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #0d0d0f;
  color: white;
  font-family: Arial, sans-serif;
}

.page {
  min-height: 100vh;
  display: grid;
  place-items: center;
  padding: 24px;
}

div {
  max-width: 600px;
  width: 100%;
  background: #17171c;
  border: 1px solid #2b2b33;
  border-radius: 16px;
  padding: 32px;
}

button {
  margin-top: 20px;
  padding: 12px 18px;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  font-weight: bold;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

a {
  color: #9ecbff;
}

# STEP 9 | Run the App

In your terminal, run:

npm run dev

Then open the local link Vite gives you, probably something like: http://localhost:5173/

# Step 10 | Before Minting 

Make sure:

1. Phantom is installed.
2. Phantom is set to Devnet.
3. Your Devnet wallet has Devnet SOL.
4. Your NFT_METADATA_URI is a real public metadata JSON URL.

The next big step is creating/uploading the NFT image and metadata JSON so your NFT_METADATA_URI is real.


# STEP 11 | Create your NFT image and use Pinata 

const NFT_METADATA_URI = "https://your-public-url.com/metadata.json";

Pinata + IPFS. Pinata lets you upload files through the web app, and IPFS gives you a public content link/CID for the NFT image and metadata.
Pinata’s docs say you can upload files from the web app by clicking Add on the Files page.  

 Choose an image - png or jpeg is fine.

 Go to Pinata, create/sign into an account, then upload your image file.

After uploading, Pinata will give you an IPFS CID or gateway URL. It may look like this:

https://gateway.pinata.cloud/ipfs/bafybeihereisyourimagecid

Copy that image URL.

Create your metadata JSON file

Create a new file on your computer called:

metadata.json

Put this inside:

{
  "name": "My First Solana NFT",
  "symbol": "JOAO",
  "description": "My first Solana NFT minted on Devnet using Metaplex Umi.",
  "image": "https://gateway.pinata.cloud/ipfs/PASTE_YOUR_IMAGE_CID_HERE",
  "attributes": [
    {
      "trait_type": "Project",
      "value": "Solana NFT"
    },
    {
      "trait_type": "Network",
      "value": "Devnet"
    }
  ],
  "properties": {
    "files": [
      {
        "uri": "https://gateway.pinata.cloud/ipfs/PASTE_YOUR_IMAGE_CID_HERE",
        "type": "image/png"
      }
    ],
    "category": "image"
  }
}

Replace both of these:

https://gateway.pinata.cloud/ipfs/PASTE_YOUR_IMAGE_CID_HERE

with your real image link.


Metaplex Token Metadata uses off-chain JSON metadata to attach things like image and attributes to the NFT, while the on-chain NFT stores the metadata URI.


Upload metadata.json to Pinata

Now upload the metadata.json file to Pinata the same way.

After uploading, copy the metadata link. It may look like:

https://gateway.pinata.cloud/ipfs/bafybeihereisyourmetadatajsoncid


This is the link you need.


Paste it into your code

In src/MintNft.tsx, replace this:
const NFT_METADATA_URI = "https://your-public-url.com/metadata.json";

with your real metadata JSON link:
const NFT_METADATA_URI = "https://gateway.pinata.cloud/ipfs/bafybeihereisyourmetadatajsoncid";


Important: NFT_METADATA_URI must point to the metadata JSON, not directly to the image.

Then run:

npm run dev


Connect Phantom on Devnet, make sure you have Devnet SOL, and click Mint NFT.


After everything works, you should see something like this on your page:
NFT minted: View on Solana Explorer


When you click the Explorer link, you’ll see a new mint address on Solana Devnet. That mint address is the NFT’s unique token address.

Example result:
NFT minted successfully.

Mint address:
8xQk...abc123

Behind the scenes, your app does this:

1. Connects your Phantom wallet.
2. Uses your wallet as the signer/payer.
3. Creates a new mint account.
4. Creates Token Metadata for the NFT.
5. Links the NFT to your hosted metadata.json.
6. Stores the NFT in your connected wallet on Devnet.

The NFT’s visible info would come from your metadata file:

{
  "name": "BEST SOL NFT EVER",
  "symbol": "THEBULLBREW",
  "description": "My first Solana NFT minted on Devnet using Metaplex Umi.",
  "image": "https://gateway.pinata.cloud/ipfs/your-image-cid"
}


So in a wallet or explorer, the NFT should eventually show:

Name: BEST SL NFT EVER
Symbol: THEBULLBREW
Image: your uploaded image
Owner: your Phantom wallet
Network: Devnet

Important: because this is on Devnet, it is not a real market NFT. It is a test NFT. It proves your minting app works. Later, you can switch to Mainnet and mint a real one.























