<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CASH DRAWER </title>

<style>
body {
    background: black;
    color: white;
    font-family: Arial;
}

.container {
    max-width: 400px;
    margin: auto;
}

.row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid green;
    padding: 10px 0;
}

.note { font-size: 20px; font-weight: bold; }

.controls {
    display: flex;
    align-items: center;
    gap: 10px;
}

button { cursor: pointer; }

.circle {
    width: 35px;
    height: 35px;
    border-radius: 50%;
    border: none;
    font-size: 18px;
}

.plus { background: green; color: white; }
.minus { background: red; color: white; }

input {
    width: 60px;
    text-align: center;
}

.section input {
    width: 100%;
    padding: 8px;
    margin-top: 5px;
}

.final {
    text-align: center;
    margin-top: 20px;
    font-size: 20px;
    color: yellow;
}

.big-btn {
    width: 100%;
    padding: 12px;
    margin-top: 10px;
    border: none;
    border-radius: 8px;
    font-size: 16px;
}

.green { background: #25D366; color: white; }
.blue { background: #2196F3; color: white; }

.history {
    margin-top: 20px;
    background: #111;
    padding: 10px;
    border-radius: 8px;
    max-height: 200px;
    overflow-y: auto;
}
</style>
</head>

<body>

<div class="container">

<h2>💰 CASH DRAWER PRO</h2>

<div id="list"></div>

<div class="section">
    <label>BANK BALANCE</label>
    <input type="number" id="bank" value="0" oninput="update()">

    <label>WALLET / CSP ID BALANCE</label>
    <input type="number" id="wallet" value="0" oninput="update()">
</div>

<div class="final" id="grandTotal">TOTAL: ₹0</div>

<button class="big-btn green" onclick="shareWhatsApp()">📤 SHARE REPORT</button>
<button class="big-btn blue" onclick="saveReport()">💾 SAVE DAILY REPORT</button>

<div class="history" id="history"></div>

</div>

<script>
const notes = [500,200,100,50,20,10];
const list = document.getElementById("list");

notes.forEach(value=>{
    let row=document.createElement("div");
    row.className="row";

    row.innerHTML=`
    <div class="note">₹${value}</div>
    <div class="controls">
        <button class="circle minus" onclick="change(this,-1)">-</button>
        <input type="number" value="0" oninput="update()">
        <button class="circle plus" onclick="change(this,1)">+</button>
    </div>
    <div class="total">₹0</div>
    `;

    list.appendChild(row);
});

function change(btn,val){
    let input=btn.parentElement.querySelector("input");
    let num=parseInt(input.value)||0;
    num+=val;
    if(num<0) num=0;
    input.value=num;
    update();
}

function update(){
    let rows=document.querySelectorAll(".row");
    let grand=0;

    rows.forEach((row,i)=>{
        let count=parseInt(row.querySelector("input").value)||0;
        let total=count*notes[i];
        row.querySelector(".total").innerText="₹"+total;
        grand+=total;
    });

    let bank=parseFloat(bankInput.value)||0;
    let wallet=parseFloat(walletInput.value)||0;

    grand+=bank+wallet;

    grandTotal.innerText="TOTAL: ₹"+grand;

    saveLocal();
}

const bankInput=document.getElementById("bank");
const walletInput=document.getElementById("wallet");
const grandTotal=document.getElementById("grandTotal");

function saveLocal(){
    let data={
        notes:[],
        bank:bankInput.value,
        wallet:walletInput.value
    };

    document.querySelectorAll(".row input").forEach(i=>{
        data.notes.push(i.value);
    });

    localStorage.setItem("cashData",JSON.stringify(data));
}

function loadLocal(){
    let data=JSON.parse(localStorage.getItem("cashData"));
    if(!data) return;

    document.querySelectorAll(".row input").forEach((i,index)=>{
        i.value=data.notes[index];
    });

    bankInput.value=data.bank;
    walletInput.value=data.wallet;

    update();
}

function shareWhatsApp(){
    let text=generateReport();
    window.open("https://wa.me/?text="+encodeURIComponent(text));
}

function generateReport(){
    let rows=document.querySelectorAll(".row");
    let text="📊 DAILY REPORT\n\n";
    let grand=0;

    rows.forEach((row,i)=>{
        let count=parseInt(row.querySelector("input").value)||0;
        let total=count*notes[i];
        if(count>0){
            text+=`₹${notes[i]} x ${count} = ₹${total}\n`;
        }
        grand+=total;
    });

    let bank=parseFloat(bankInput.value)||0;
    let wallet=parseFloat(walletInput.value)||0;

    text+=`\nBANK: ₹${bank}`;
    text+=`\nWALLET/CSP: ₹${wallet}`;

    grand+=bank+wallet;

    text+=`\n\nTOTAL: ₹${grand}`;

    return text;
}

function saveReport(){
    let history=JSON.parse(localStorage.getItem("history"))||[];

    let report={
        date:new Date().toLocaleString(),
        text:generateReport()
    };

    history.unshift(report);
    localStorage.setItem("history",JSON.stringify(history));

    loadHistory();
}

function loadHistory(){
    let history=JSON.parse(localStorage.getItem("history"))||[];
    let box=document.getElementById("history");

    box.innerHTML="<b>📅 HISTORY</b><br>";

    history.forEach(h=>{
        box.innerHTML+=`<hr>${h.date}<br><pre>${h.text}</pre>`;
    });
}

loadLocal();
loadHistory();
</script>

</body>
</html>
