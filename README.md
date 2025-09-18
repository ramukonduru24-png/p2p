<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Document-Backed Lending DApp</title>
  <script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <script src="https://unpkg.com/@lottiefiles/lottie-player@latest/dist/lottie-player.js"></script>

  <style>
    :root {
      --bg-light: #f3f4f6;
      --bg-dark: #121212;
      --text-light: #1f2937;
      --text-dark: #f9fafb;
    }

    body {
      background: linear-gradient(to right, #e0f7fa, #ede7f6);
      transition: all 0.4s ease-in-out;
      font-family: 'Segoe UI', sans-serif;
    }

    body.dark-mode {
      background: linear-gradient(to right, #121212, #1f1f1f);
      color: var(--text-dark);
    }

    .container {
      background: rgba(255, 255, 255, 0.85);
      border-radius: 20px;
      box-shadow: 0 8px 25px rgba(0, 0, 0, 0.1);
      transition: 0.4s;
    }

    body.dark-mode .container {
      background: rgba(33, 33, 33, 0.85);
      color: var(--text-dark);
    }

    .title {
      font-size: 2.7rem;
      font-weight: bold;
      text-align: center;
      background: linear-gradient(to right, #8e44ad, #3498db);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }

    .lender-panel {
      background: linear-gradient(135deg, #e3f2fd, #bbdefb);
      border-radius: 20px;
      padding: 1rem;
    }

    .borrower-panel {
      background: linear-gradient(135deg, #f3e5f5, #ce93d8);
      border-radius: 20px;
      padding: 1rem;
    }

    .card {
      border-radius: 20px;
      backdrop-filter: blur(8px);
      box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
      transition: 0.3s ease-in-out;
    }

    .card:hover {
      transform: translateY(-5px);
      box-shadow: 0 12px 30px rgba(0, 0, 0, 0.2);
    }

    .form-control {
      border-radius: 12px;
    }

    .btn {
      border-radius: 12px;
      font-weight: 600;
      transition: 0.3s;
    }

    .btn-success { background: linear-gradient(to right, #10b981, #34d399); color: white; border: none; }
    .btn-warning { background: linear-gradient(to right, #f59e0b, #fbbf24); color: white; border: none; }
    .btn-primary { background: linear-gradient(to right, #3b82f6, #60a5fa); color: white; border: none; }
    .btn-purple  { background: linear-gradient(to right, #8b5cf6, #a78bfa); color: white; border: none; }
    .btn-reset   { background: linear-gradient(to right, #14b8a6, #2dd4bf); color: white; border: none; }

    #repayStatus {
      font-weight: bold;
    }

    .dark-toggle {
      position: fixed;
      top: 1rem;
      right: 1rem;
      z-index: 999;
    }

    /* Toast styles */
    #toastBox {
      position: fixed;
      bottom: 20px;
      right: 20px;
      z-index: 9999;
    }

    .toast {
      background-color: #0d6efd;
      color: white;
      padding: 12px 20px;
      border-radius: 10px;
      margin-top: 10px;
      animation: slideIn 0.5s ease-out;
    }

    @keyframes slideIn {
      from { opacity: 0; transform: translateX(100px); }
      to { opacity: 1; transform: translateX(0); }
    }
  </style>
</head>
<body>
  <button class="btn btn-dark dark-toggle" onclick="toggleDarkMode()">🌓 Toggle Dark Mode</button>

  <div class="container py-5 px-4">
    <h2 class="title mb-4">📜 Document-Backed Lending DApp</h2>

    <lottie-player
      src="https://assets5.lottiefiles.com/packages/lf20_ydo1amjm.json"
      background="transparent"
      speed="1"
      style="width: 100px; height: 100px; margin: 0 auto;"
      loop
      autoplay>
    </lottie-player>

    <div class="text-end mb-3">
      <button onclick="resetLoanManually()" class="btn btn-reset btn-sm">🔁 Reset Loan (Lender Only)</button>
    </div>

    <div class="card mb-4 p-4">
      <h5>🔌 Wallet Connection</h5>
      <button class="btn btn-primary mb-2" onclick="connectWallet()">Connect Wallet</button>
      <p id="walletAddress" class="text-success"></p>
    </div>

    <div class="row">
      <div class="col-md-6 mb-4">
        <div class="lender-panel h-100">
          <div class="card p-4">
            <h5>💰 Lender Panel</h5>
            <input id="interestRate" class="form-control mb-2" placeholder="Interest Rate (%)">
            <input id="loanAmount" class="form-control mb-2" placeholder="Loan Amount (in ETH)">
            <input type="file" id="lenderDoc" class="form-control mb-3">
            <button class="btn btn-success w-100" onclick="offerLoan()">Offer Loan</button>
          </div>
        </div>
      </div>

      <div class="col-md-6 mb-4">
        <div class="borrower-panel h-100">
          <div class="card p-4">
            <h5>📄 Borrower Panel</h5>
            <input type="file" id="borrowerDoc" class="form-control mb-3">
            <button class="btn btn-warning w-100 mb-2" onclick="submitDocument()">Submit Document</button>
            <button class="btn btn-primary w-100 mb-2" onclick="takeLoan()">Take Loan</button>
            <button class="btn btn-purple w-100 mb-2" onclick="repayLoan()">Repay Loan</button>
            <p id="repayStatus" class="mt-3 text-danger"></p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div id="toastBox"></div>

  <script>
    // Toggle Dark Mode
    function toggleDarkMode() {
      document.body.classList.toggle("dark-mode");
    }

    // Toast Notification
    function showToast(message, type = "info") {
      const toast = document.createElement("div");
      toast.className = "toast";
      toast.innerText = message;

      if (type === "error") toast.style.backgroundColor = "#dc3545";
      if (type === "success") toast.style.backgroundColor = "#28a745";

      document.getElementById("toastBox").appendChild(toast);
      setTimeout(() => toast.remove(), 3000);
    }

let web3;
let account;

// ✅ Contract Address
const contractAddress = "0xC2e0620752E4115cA679d71Ad2c9ee63f2d046Fc";

// ✅ Contract ABI
const contractABI = [
	{
		"inputs":[{"internalType":"uint256","name":"_interestRate","type":"uint256"},{"internalType":"bytes32","name":"_docHash","type":"bytes32"}],
		"name":"offerLoan","outputs":[],"stateMutability":"payable","type":"function"
	},
	{"inputs":[],"name":"repayLoan","outputs":[],"stateMutability":"payable","type":"function"},
	{"inputs":[],"name":"resetLoan","outputs":[],"stateMutability":"nonpayable","type":"function"},
	{"inputs":[{"internalType":"bytes32","name":"_userDocHash","type":"bytes32"}],
		"name":"submitDocument","outputs":[],"stateMutability":"nonpayable","type":"function"},
	{"inputs":[],"name":"takeLoan","outputs":[],"stateMutability":"nonpayable","type":"function"},
	{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},
	{"inputs":[],"name":"borrower","outputs":[{"internalType":"address payable","name":"","type":"address"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"docSubmitted","outputs":[{"internalType":"bool","name":"","type":"bool"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"documentHash","outputs":[{"internalType":"bytes32","name":"","type":"bytes32"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"interestRate","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"lender","outputs":[{"internalType":"address payable","name":"","type":"address"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"loanAmount","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"loanOffered","outputs":[{"internalType":"bool","name":"","type":"bool"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"loanRepaid","outputs":[{"internalType":"bool","name":"","type":"bool"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"loanStartTime","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],
		"stateMutability":"view","type":"function"},
	{"inputs":[],"name":"loanTaken","outputs":[{"internalType":"bool","name":"","type":"bool"}],
		"stateMutability":"view","type":"function"}
];

let contract;

async function connectWallet() {
  if (window.ethereum) {
    web3 = new Web3(window.ethereum);
    await window.ethereum.request({ method: 'eth_requestAccounts' });
    const accounts = await web3.eth.getAccounts();
    account = accounts[0];
    document.getElementById("walletAddress").innerText = Connected: ${account};
    contract = new web3.eth.Contract(contractABI, contractAddress);
    showToast("Wallet connected!", "success");
  } else {
    showToast("Please install MetaMask", "error");
  }
}

async function offerLoan() {
  const rate = document.getElementById("interestRate").value;
  const amount = document.getElementById("loanAmount").value;
  const file = document.getElementById("lenderDoc").files[0];
  if (!file) return showToast("Please upload a document", "error");

  const hash = await getFileHash(file);
  try {
    await contract.methods.offerLoan(rate, hash).send({ from: account, value: web3.utils.toWei(amount, 'ether') });
    showToast("Loan offered successfully.", "success");
  } catch (err) {
    console.error(err);
    showToast("Failed to offer loan. Reset first and try again.", "error");
  }
}

async function submitDocument() {
  const file = document.getElementById("borrowerDoc").files[0];
  if (!file) return showToast("Upload the same document", "error");

  const hash = await getFileHash(file);
  await contract.methods.submitDocument(hash).send({ from: account });
  showToast("Document submitted", "success");
}

async function takeLoan() {
  await contract.methods.takeLoan().send({ from: account });
  showToast("Loan taken successfully", "success");
}

async function repayLoan() {
  try {
    const [loanAmount, interestRate, loanStartTime, lenderAddress] = await Promise.all([
      contract.methods.loanAmount().call(),
      contract.methods.interestRate().call(),
      contract.methods.loanStartTime().call(),
      contract.methods.lender().call()
    ]);

    const now = Math.floor(Date.now() / 1000);
    const elapsed = now - Number(loanStartTime);
    const monthsElapsed = Math.floor(elapsed / (30 * 24 * 60 * 60));

    const interest = (BigInt(loanAmount) * BigInt(interestRate) * BigInt(monthsElapsed + 1)) / 100n;
    const totalDue = BigInt(loanAmount) + interest;

    await contract.methods.repayLoan().send({ from: account, value: totalDue.toString() });

    document.getElementById("repayStatus").innerText = "✅ Repayment successful.";
    document.getElementById("repayStatus").classList.remove("text-danger");
    document.getElementById("repayStatus").classList.add("text-success");

    if (account.toLowerCase() === lenderAddress.toLowerCase()) {
      try {
        await contract.methods.resetLoan().send({ from: account });
        document.getElementById("repayStatus").innerText = "✅ Loan repaid and reset.";
      } catch (resetError) {
        console.error("Reset failed:", resetError);
        document.getElementById("repayStatus").innerText = "Repayment done, but loan reset failed.";
      }
    } else {
      document.getElementById("repayStatus").innerText = "✅ Repayment done. Lender must reset to start a new loan.";
    }
  } catch (err) {
    console.error("Repay failed:", err);
    document.getElementById("repayStatus").innerText = "❌ Failed to repay loan.";
  }
}

async function resetLoanManually() {
  try {
    await contract.methods.resetLoan().send({ from: account });
    document.getElementById("repayStatus").innerText = "Loan reset manually by lender.";
    document.getElementById("repayStatus").classList.remove("text-danger");
    document.getElementById("repayStatus").classList.add("text-success");
  } catch (err) {
    console.error(err);
    document.getElementById("repayStatus").innerText = "Manual reset failed.";
  }
}

// ✅ Fixed Hash Function
async function getFileHash(file) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = async () => {
      try {
        const buffer = reader.result; // ArrayBuffer
        const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
        const hashArray = Array.from(new Uint8Array(hashBuffer));
        const hashHex = "0x" + hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
        resolve(hashHex);
      } catch (err) {
        reject(err);
      }
    };
    reader.onerror = reject;
    reader.readAsArrayBuffer(file);
  });
}
  </script>
</body>
</html>
