Các bước cần thực hiện:
- Tạo bot telegram
Lấy Token
Lấy chat ID
Nhập thông tin vào file code
Tạo file Excel hoặc CSV theo mẫu
Điền list IP cần kiểm tra vào file
Chạy thôi
các đoạn code bên dưới, vì không up nhiều file được
PYTHON
#####Cài đặt thư viện
pip install pandas requests openpyxl
import requests
import pandas as pd
from datetime import datetime
import os
import platform
import subprocess
#################### Thông tin Bot
telegram_token = "token ở đây"
telegram_chat_id = "chat ID ở đây"
#################### Hàm gửi tin nhắn Telegram
def send_telegram_message(message):
url = f"https:#########################api.telegram.org/bot{telegram_token}/sendMessage"
params = {
"chat_id": telegram_chat_id,
"text": message
}
response = requests.post(url, data=params)
return response
#################### Hàm ping IP
def ping_host(ip):
param = "-n" if platform.system().lower() == "windows" else "-c"
command = ["ping", param, "1", ip]
result = subprocess.run(command, stdout=subprocess.PIPE)
if result.returncode == 0:
return "Online"
else:
return "Offline"
#################### Đường dẫn đến file Excel
excel_file = "D:/APP_TEST/IPList.xlsx"
#################### Đọc file Excel
workbook = pd.read_excel(excel_file)
#################### Duyệt qua từng dòng trong Excel (bỏ qua dòng tiêu đề)
for index, row in workbook.iterrows():
ip = row["IP"]
name = row["Name"]
status = ping_host(ip)
timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
#################### Cập nhật trạng thái và thời gian
workbook.at[index, "Status"] = status
workbook.at[index, "Timestamp"] = timestamp
#################### Nếu IP là Offline, gửi tin nhắn báo qua Telegram
if status == "Offline":
message = f"ALERT: IP {ip} - {name} is OFFLINE at {timestamp}"
send_telegram_message(message)
#################### Lưu lại thay đổi vào file Excel
workbook.to_excel(excel_file, index=False)
POWERSHELL
#################### Cho phép chạy PW trước
#####Set-ExecutionPolicy RemoteSigned -Scope Process
####################Cài đặt Excel Module nếu chưa có
#####Install-Module -Name ImportExcel -Force
#################### Thông tin Telegram Bot
$telegramToken = "5597477930:AAG9ugOKMbbtiFrOiXfMKXR_LHCkHBJar6Q"
$telegramChatId = "5744415860"
#################### Hàm gửi tin nhắn Telegram
function Send-TelegramMessage {
param (
[string]$message
)
$url = "https:#########################api.telegram.org/bot$telegramToken/sendMessage"
$params = @{
chat_id = $telegramChatId
text = $message
}
Invoke-RestMethod -Uri $url -Method Post -Body $params
}
#################### Hàm ping IP
function Ping-Host {
param (
[string]$ip
)
$pingResult = Test-Connection -ComputerName $ip -Count 1 -Quiet
if ($pingResult) {
return "Online"
}
else {
return "Offline"
}
}
#################### Đường dẫn đến file Excel
$excelFile = "D:\APP_TEST\IPList.xlsx"
######################### Đọc file Excel
$workbook = Import-Excel -Path $excelFile
######################### Duyệt qua từng dòng trong Excel (bỏ qua dòng tiêu đề)
foreach ($row in $workbook | Where-Object { $_.IP }) {
$ip = $row.IP
$name=$row.Name
$status = Ping-Host -ip $ip
$timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
######################### Cập nhật trạng thái và thời gian
$row.Status = $status
$row.Timestamp = $timestamp
######################### Nếu IP là Offline, gửi tin nhắn báo qua Telegram
if ($status -eq "Offline") {
$message = "ALERT: IP $ip - $name is OFFLINE at $timestamp "
Send-TelegramMessage -message $message
}
}
############################## Lưu lại thay đổi vào file Excel
$workbook | Export-Excel -Path $excelFile -WorksheetName "Sheet1"
JAVASCRIPT
#########################Cài đăth thư viện
npm install axios ping exceljs
const axios = require('axios');
const ping = require('ping');
const ExcelJS = require('exceljs');
const path = require('path');
##### ####################Thông tin Telegram Bot
const telegramToken = "đây là access token";
const telegramChatId = "đây là chat ID";
######################### Hàm gửi tin nhắn Telegram
async function sendTelegramMessage(message) {
const url = `https:#########################api.telegram.org/bot${telegramToken}/sendMessage`;
const params = {
chat_id: telegramChatId,
text: message
};
try {
const response = await axios.post(url, params);
return response.data;
} catch (error) {
console.error("Error sending message to Telegram", error);
}
}
##### Hàm ping IP
async function pingHost(ip) {
try {
const res = await ping.promise.probe(ip, { timeout: 2 });
return res.alive ? "Online" : "Offline";
} catch (error) {
console.error(`Error pinging IP ${ip}`, error);
return "Offline";
}
}
##### Đường dẫn đến file Excel
const excelFile = path.resolve(__dirname, 'IPList.xlsx');
##### Đọc file Excel và xử lý các hàng
async function processExcelFile() {
const workbook = new ExcelJS.Workbook();
await workbook.xlsx.readFile(excelFile);
const worksheet = workbook.getWorksheet(1);
######################### Duyệt qua từng dòng trong Excel (bỏ qua dòng tiêu đề)
for (let rowIndex = 2; rowIndex <= worksheet.rowCount; rowIndex++) {
const row = worksheet.getRow(rowIndex);
const ip = row.getCell(1).value; ######################### Cột IP
const name = row.getCell(2).value; ######################### Cột Name
const status = await pingHost(ip);
const timestamp = new Date().toISOString().replace('T', ' ').substring(0, 19); ######################### Format yyyy-MM-dd HH:mm:ss
######################### Cập nhật trạng thái và thời gian
row.getCell(3).value = status; ######################### Cột Status
row.getCell(4).value = timestamp; ######################### Cột Timestamp
######################### Nếu IP là Offline, gửi tin nhắn báo qua Telegram
if (status === "Offline") {
const message = `ALERT: IP ${ip} - ${name} is OFFLINE at ${timestamp}`;
await sendTelegramMessage(message);
}
row.commit(); ######################### Ghi lại thay đổi trong dòng
}
######################### Lưu lại thay đổi vào file Excel
await workbook.xlsx.writeFile(excelFile);
}
######################### Chạy chương trình
processExcelFile().then(() => {
console.log("Processing completed.");
}).catch((error) => {
console.error("Error processing Excel file", error);
});
C#
#####Cài đặt package
Install-Package EPPlus
using System;
using System.IO;
using System.Net.Http;
using System.Net.NetworkInformation;
using System.Threading.Tasks;
using OfficeOpenXml;
class Program
{
######################### Thông tin Telegram Bot
static string telegramToken = "đây là access token";
static string telegramChatId = "đây là chat ID";
static async Task Main(string[] args)
{
######################### Đường dẫn đến file Excel
string excelFile = @"D:\APP_TEST\IPList.xlsx";
######################### Đọc file Excel
FileInfo fileInfo = new FileInfo(excelFile);
ExcelPackage.LicenseContext = LicenseContext.NonCommercial;
using (var package = new ExcelPackage(fileInfo))
{
ExcelWorksheet worksheet = package.Workbook.Worksheets[0]; ######################### dữ liệu ở Sheet1
int rowCount = worksheet.Dimension.Rows;
######################### Duyệt qua từng dòng trong Excel (bỏ qua dòng tiêu đề)
for (int row = 2; row <= rowCount; row++)
{
string ip = worksheet.Cells[row, 1].Text; ######################### Cột IP
string name = worksheet.Cells[row, 2].Text; ######################### Cột Name
string status = PingHost(ip);
string timestamp = DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss");
######################### Cập nhật trạng thái và thời gian
worksheet.Cells[row, 3].Value = status; ######################### Cột Status
worksheet.Cells[row, 4].Value = timestamp; ######################### Cột Timestamp
######################### Nếu IP là Offline, gửi tin nhắn báo qua Telegram
if (status == "Offline")
{
string message = $"ALERT: IP {ip} - {name} is OFFLINE at {timestamp}";
await SendTelegramMessage(message);
}
}
######################### Lưu lại thay đổi vào file Excel
package.Save();
}
Console.WriteLine("Processing completed.");
}
######################### Hàm gửi tin nhắn Telegram
static async Task SendTelegramMessage(string message)
{
string url = $"https:#########################api.telegram.org/bot{telegramToken}/sendMessage";
using (var client = new HttpClient())
{
var content = new FormUrlEncodedContent(new[]
{
new KeyValuePair<string, string>("chat_id", telegramChatId),
new KeyValuePair<string, string>("text", message)
});
HttpResponseMessage response = await client.PostAsync(url, content);
if (!response.IsSuccessStatusCode)
{
Console.WriteLine($"Error sending message to Telegram: {response.StatusCode}");
}
}
}
######################### Hàm ping IP
static string PingHost(string ip)
{
using (Ping ping = new Ping())
{
try
{
PingReply reply = ping.Send(ip, 1000); ######################### Timeout là 1000ms
if (reply.Status == IPStatus.Success)
{
return "Online";
}
else
{
return "Offline";
}
}
catch (Exception ex)
{
Console.WriteLine($"Error pinging {ip}: {ex.Message}");
return "Offline";
}
}
}
}
