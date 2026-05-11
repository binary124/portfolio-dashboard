  
<!DOCTYPE html>  
<html>  
<head>  
  <title>Portfolio Dashboard</title>  
  <style>  
    body { font-family: Arial; padding: 20px; }  
    table { border-collapse: collapse; width: 100%; margin-top: 20px; }  
    th, td { border: 1px solid #ccc; padding: 10px; text-align: center; }  
    button { padding: 5px 10px; margin: 5px; }  
    input { padding: 5px; margin: 5px; }  
  </style>  
</head>  
  
<body>  
  
<h1>📊 My Portfolio Dashboard</h1>  
  
<div>  
  <input id="symbol" placeholder="Symbol (e.g. RELIANCE.NS)">  
  <input id="qty" type="number" placeholder="Quantity">  
  <input id="price" type="number" placeholder="Buy Price">  
  <button onclick="addStock()">Add</button>  
  <button onclick="exportCSV()">Export CSV</button>  
</div>  
  
<table id="portfolioTable">  
  <thead>  
    <tr>  
      <th>Symbol</th>  
      <th>Qty</th>  
      <th>Buy Price</th>  
      <th>Live Price</th>  
      <th>P&L</th>  
      <th>Action</th>  
    </tr>  
  </thead>  
  <tbody></tbody>  
</table>  
  
<h2 id="totalPnL">Total P&L: ₹0</h2>  
  
<script>  
let portfolio = JSON.parse(localStorage.getItem("portfolio")) || [];  
  
async function fetchPrice(symbol) {  
  try {  
    let res = await fetch(`https://query1.finance.yahoo.com/v7/finance/quote?symbols=${symbol}`);  
    let data = await res.json();  
    return data.quoteResponse.result[0].regularMarketPrice || 0;  
  } catch {  
    return 0;  
  }  
}  
  
function saveData() {  
  localStorage.setItem("portfolio", JSON.stringify(portfolio));  
}  
  
function addStock() {  
  const symbol = document.getElementById("symbol").value;  
  const qty = parseFloat(document.getElementById("qty").value);  
  const price = parseFloat(document.getElementById("price").value);  
  
  if (!symbol || !qty || !price) return alert("Fill all fields");  
  
  portfolio.push({ symbol, qty, price });  
  saveData();  
  render();  
}  
  
function deleteStock(index) {  
  portfolio.splice(index, 1);  
  saveData();  
  render();  
}  
  
async function render() {  
  const tbody = document.querySelector("#portfolioTable tbody");  
  tbody.innerHTML = "";  
  
  let totalPnL = 0;  
  
  for (let i = 0; i < portfolio.length; i++) {  
    let stock = portfolio[i];  
    let live = await fetchPrice(stock.symbol);  
    let pnl = (live - stock.price) * stock.qty;  
  
    totalPnL += pnl;  
  
    let row = `<tr>  
      <td>${stock.symbol}</td>  
      <td>${stock.qty}</td>  
      <td>${stock.price}</td>  
      <td>${live.toFixed(2)}</td>  
      <td>${pnl.toFixed(2)}</td>  
      <td><button onclick="deleteStock(${i})">Delete</button></td>  
    </tr>`;  
  
    tbody.innerHTML += row;  
  }  
  
  document.getElementById("totalPnL").innerText =  
    "Total P&L: ₹" + totalPnL.toFixed(2);  
}  
  
function exportCSV() {  
  let csv = "Symbol,Qty,Buy Price\n";  
  portfolio.forEach(s => {  
    csv += `${s.symbol},${s.qty},${s.price}\n`;  
  });  
  
  let blob = new Blob([csv], { type: "text/csv" });  
  let a = document.createElement("a");  
  a.href = URL.createObjectURL(blob);  
  a.download = "portfolio.csv";  
  a.click();  
}  
  
render();  
</script>  
  
</body>  
</html>  
