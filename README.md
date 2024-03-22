- 👋 Hi, I’m @kaushal7488
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
kaushal7488/kaushal7488 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
//@version=5
indicator("4Min Binary Strategy", overlay=true)

DoAlert = input(true, title="Enable Alerts")
var int CurrentState = 0
var float Price = na
var int LastTm = na
var int Total = 0
var int TotalSuccess = 0

// Custom indicator conditions
smaCrossOver = ta.crossover(ta.sma(close, 50), ta.sma(close, 200))
adxCondition = ta.crossover(ta.adx(close, 14), 25)
adx2Condition = ta.crossover(ta.adx(close, 14), 35)

// Main strategy logic
if (smaCrossOver)
    if (adxCondition)
        if (adx2Condition)
            if (CurrentState == 0)
                CurrentState := 1
                Price := close
                if (DoAlert)
                    if (LastTm < time)
                        alert("CALL: " + syminfo.ticker + " : " + str.tostring(close, '#.######') + " : EXPIRY: " + str.tostring(time + 300))
                LastTm := time

if (not smaCrossOver)
    if (adxCondition)
        if (adx2Condition)
            if (CurrentState == 0)
                CurrentState := -1
                Price := close
                if (DoAlert)
                    if (LastTm < time)
                        alert("PUT: " + syminfo.ticker + " : " + str.tostring(close, '#.######') + " : EXPIRY: " + str.tostring(time + 300))
                LastTm := time

// Check previous state for success/failure
if (CurrentState == 1)
    if (LastTm == ta.barssince(smaCrossOver))
        if (close[1] > Price)
            TotalSuccess := TotalSuccess + 1
        else
            Total := Total + 1
        CurrentState := 0

if (CurrentState == -1)
    if (LastTm == ta.barssince(not smaCrossOver))
        if (close[1] < Price)
            TotalSuccess := TotalSuccess + 1
        else
            Total := Total + 1
        CurrentState := 0

// Display information
DoDisplay() =>
    var string s = ""
    var float tmp = na
    s := "\n5 Minute Binary Strategy\n"
    s := s + "===============\n\n"
    s := s + (DoAlert ? "ALERT: ON\n" : "ALERT: OFF\n")
    s := s + (CurrentState == 0 ? "STATE: WAITING\n\n" : na)
    s := s + (CurrentState == 1 ? "STATE: CALL\n\n" : na)
    s := s + (CurrentState == -1 ? "STATE: PUT\n\n" : na)
    s := s + "TOTAL TRADES: " + str.tostring(Total) + "\n"
    s := s + "TOTAL WINS: " + str.tostring(TotalSuccess) + "\n"
    if (Total > 0)
        tmp := TotalSuccess
        tmp := tmp / Total
        s := s + "SUCCESS PERCENTAGE: " + str.tostring(tmp * 100, '#.##') + "\n"
    label.new(bar_index, na, text=s, yloc=yloc.belowbar)

DoDisplay()

// Your existing indicator logic
plot(close)
