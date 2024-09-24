- 👋 Hi, I’m @dapur aset
- 👀 I’m interested in ...pengembangan blockchain
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ..
Landing dan Borrowing Platform:memingkinkan pengguna meminjam atau meminjamkan aset tanpa memerlukan lembaga perantara
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract LendingBorrowing {
    struct Loan {
        address borrower;
        uint amount;
        bool isActive;
    }

    mapping(address => uint) public balances;
    mapping(uint => Loan) public loans;
    uint public loanCount;

    event Deposited(address indexed user, uint amount);
    event LoanRequested(uint indexed loanId, address indexed borrower, uint amount);
    event LoanPaid(uint indexed loanId);

    // Deposit funds into the contract
    function deposit() public payable {
        require(msg.value > 0, "Deposit amount must be greater than 0");
        balances[msg.sender] += msg.value;
        emit Deposited(msg.sender, msg.value);
    }

    // Request a loan
    function requestLoan(uint amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        loans[loanCount] = Loan(msg.sender, amount, true);
        loanCount++;
        emit LoanRequested(loanCount - 1, msg.sender, amount);
    }

    // Pay back the loan
    function payLoan(uint loanId) public {
        Loan storage loan = loans[loanId];
        require(loan.isActive, "Loan is not active");
        require(msg.sender == loan.borrower, "Only borrower can pay back the loan");

        balances[msg.sender] -= loan.amount;
        loan.isActive = false;
        emit LoanPaid(loanId);
    }

    // Withdraw funds
    function withdraw(uint amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }
}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lending and Borrowing DApp</title>
    <script src="https://cdn.jsdelivr.net/npm/web3/dist/web3.min.js"></script>
</head>
<body>
    <h1>Lending and Borrowing DApp</h1>
    <div>
        <h2>Deposit</h2>
        <input type="text" id="depositAmount" placeholder="Amount to deposit">
        <button onclick="deposit()">Deposit</button>
    </div>
    <div>
        <h2>Request Loan</h2>
        <input type="text" id="loanAmount" placeholder="Amount to request">
        <button onclick="requestLoan()">Request Loan</button>
    </div>
    <div>
        <h2>Pay Loan</h2>
        <input type="text" id="loanId" placeholder="Loan ID">
        <button onclick="payLoan()">Pay Loan</button>
    </div>
    <script>
        const contractAddress = 'YOUR_CONTRACT_ADDRESS';
        const abi = [ /* ABI dari contract di atas */ ];
        let web3;
        let contract;

        async function init() {
            if (window.ethereum) {
                web3 = new Web3(window.ethereum);
                await window.ethereum.enable();
                contract = new web3.eth.Contract(abi, contractAddress);
            } else {
                alert('Please install MetaMask!');
            }
        }

        async function deposit() {
            const amount = document.getElementById('depositAmount').value;
            const accounts = await web3.eth.getAccounts();
            await contract.methods.deposit().send({ from: accounts[0], value: web3.utils.toWei(amount, 'ether') });
        }

        async function requestLoan() {
            const amount = document.getElementById('loanAmount').value;
            const accounts = await web3.eth.getAccounts();
            await contract.methods.requestLoan(web3.utils.toWei(amount, 'ether')).send({ from: accounts[0] });
        }

        async function payLoan() {
            const loanId = document.getElementById('loanId').value;
            const accounts = await web3.eth.getAccounts();
            await contract.methods.payLoan(loanId).send({ from: accounts[0] });
        }

        window.onload = init;
    </script>
</body>
</html>
