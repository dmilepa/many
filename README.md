// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © LonesomeTheBlue
//
//@version=4
study("Divergence for many indicator v3", overlay=true, max_bars_back = 4000)
lb = input(5, title="Left Bars", minval=1)
rb = input(5, title="Right Bars", minval=1)
showhidden = input(false, title = "Show Hidden Divergences")
chcut = input(true, title = "Check Cut-Through in indicators !")
shownum = input(true, title="Show Divergence Number")
showindis = input(false, title="Show Indicator Names")
showpivot = input(false, title="Show Pivot Points")
chwidth = input(true, title = "Change Width by Number of Divergence")
showlimit = input(1, title="Minimum Number of Divergence", minval = 1, maxval = 11)
calcmacd = input(true, title="MACD")
calcmacda = input(true, title="MACD Histogram")
calcrsi = input(true, title="RSI")
calcstoc = input(true, title="Stochastic")
calccci = input(true, title="CCI")
calcmom = input(true, title="Momentum")
calcobv = input(true, title="OBV")
calcdi = input(true, title="Diosc")
calcvwmacd = input(true, title="VWmacd")
calccmf = input(true, title="Chaikin Money Flow")
calcmfi = input(true, title="Money Flow Index")

// RSI
rsi = rsi(close, 14)
// MACD
[macd, signal, deltamacd] = macd(close, 12, 26, 9)
// Momentum
moment = mom(close, 10)
// CCI
cci = cci(close, 10)
// OBV

Obv = obv // cum(change(close) > 0 ? volume : change(close) < 0 ? -volume : 0 * volume)
// Stoch
stk = sma(stoch(close, high, low, 14), 3)
// DIOSC
DI = change(high) - (-change(low))
trur = rma(tr, 14)
diosc = fixnan(100 * rma(DI, 14) / trur)
// volume weighted macd
maFast = vwma(close, 12)
maSlow = vwma(close, 26)
vwmacd = maFast - maSlow
// Chaikin money flow
Cmfm = ((close-low) - (high-close)) / (high - low)
Cmfv = Cmfm * volume
cmf = sma(Cmfv, 21) / sma(volume,21)
// Moneyt Flow Index
Mfi = mfi(close, 14)

float top = na
float bot = na
top := pivothigh(lb, rb)
bot := pivotlow(lb, rb)

plotshape(top and showpivot, text="[PH]",  style=shape.labeldown, color=color.white, textcolor=color.black, location=location.abovebar, offset = -rb)
plotshape(bot and showpivot, text="[PL]",  style=shape.labeldown, color=color.white, textcolor=color.black, location=location.belowbar,  offset = -rb)

topc = 0, botc = 0
topc := top ? lb : nz(topc[1]) + 1
botc := bot ? lb : nz(botc[1]) + 1

// Regular Negative /hidden Divergences
newtop = pivothigh(lb, 0) // check only left side
emptyh = true
if not na(newtop) and ((newtop > high[topc] and not showhidden) or (newtop < high[topc] and showhidden))  // there must not close price higher than the line between last PH and current high
    diff = (newtop - high[topc]) / topc
    hline = newtop - diff                   // virtual line to check there is no close price higher than it
    for x = 1 to topc -1
        if close[x] > hline
            emptyh := false
            break
        hline := hline - diff
else
    emptyh := false

// check cut-through in indicators
nocut1(indi, len)=>
    _ret = true
    diff = (indi - nz(indi[len])) / len
    ln = indi - diff
    for x = 1 to len -1
        if nz(indi[x]) > ln
            _ret := false
            break
        ln := ln - diff
    _ret

rsiok = nocut1(rsi, topc)
macdok = nocut1(macd, topc)
deltamacdok = nocut1(deltamacd, topc)
momentok = nocut1(moment, topc)
cciok = nocut1(cci, topc)
obvok = nocut1(obv, topc)
stkok = nocut1(stk, topc)
dioscok = nocut1(diosc, topc)
vwmacdok = nocut1(vwmacd, topc)
cmfok = nocut1(cmf, topc)
mfiok = nocut1(Mfi, topc)

// Regular Negative Divergences
negdivergence = 0
negdivtxt = ""
if emptyh and not na(newtop) and not showhidden
    if calcrsi and rsi[topc] > rsi and (not chcut or rsiok)
        negdivergence += 1
        negdivtxt := "RSI\n"
    if calcmacd and macd[topc] > macd and (not chcut or macdok)
        negdivergence += 1
        negdivtxt := negdivtxt + "MACD\n"
    if calcmacda and deltamacd[topc] > deltamacd and (not chcut or deltamacdok)
        negdivergence += 1
        negdivtxt := negdivtxt + "MACD Hist\n"
    if calcmom and moment[topc] > moment and (not chcut or momentok)
        negdivergence += 1
        negdivtxt := negdivtxt + "Momentum\n"
    if calccci and cci[topc] > cci and (not chcut or cciok)
        negdivergence := negdivergence + 1
        negdivtxt := negdivtxt + "CCI\n"
    if calcobv and Obv[topc] > Obv and (not chcut or obvok)
        negdivergence += 1
        negdivtxt := negdivtxt + "OBV\n"
    if calcstoc and stk[topc] > stk and (not chcut or stkok)
        negdivergence += 1
        negdivtxt := negdivtxt + "Stoch\n"
    if calcdi and diosc[topc] > diosc and (not chcut or dioscok)
        negdivergence += 1
        negdivtxt := negdivtxt + "Diosc\n"
    if calcvwmacd and vwmacd[topc] > vwmacd and (not chcut or vwmacdok)
        negdivergence += 1
        negdivtxt := negdivtxt + "VWMacd\n"
    if calccmf and cmf[topc] > cmf  and (not chcut or cmfok)
        negdivergence += 1
        negdivtxt := negdivtxt + "CMF\n"
    if calcmfi and Mfi[topc] > Mfi  and (not chcut or mfiok)
        negdivergence += 1
        negdivtxt := negdivtxt + "MFI\n"

// negative hidden divergence
hnegdivergence = 0
hnegdivtxt = ""
if emptyh and not na(newtop) and showhidden
    if calcrsi and rsi[topc] < rsi and (not chcut or rsiok)
        hnegdivergence += 1
        hnegdivtxt := "RSI\n"
    if calcmacd and macd[topc] < macd and (not chcut or macdok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "MACD\n"
    if calcmacda and deltamacd[topc] < deltamacd and (not chcut or deltamacdok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "MACD Hist\n"
    if calcmom and moment[topc] < moment and (not chcut or momentok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "Momentum\n"
    if calccci and cci[topc] < cci and (not chcut or cciok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "CCI\n"
    if calcobv and Obv[topc] < Obv and (not chcut or obvok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "OBV\n"
    if calcstoc and stk[topc] < stk and (not chcut or stkok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "Stoch\n"
    if calcdi and diosc[topc] < diosc and (not chcut or dioscok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "Diosc\n"
    if calcvwmacd and vwmacd[topc] < vwmacd and (not chcut or vwmacdok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "VWMacd\n"
    if calccmf and cmf[topc] < cmf and (not chcut or cmfok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "CMF\n"
    if calcmfi and Mfi[topc] < Mfi  and (not chcut or mfiok)
        hnegdivergence += 1
        hnegdivtxt := hnegdivtxt + "MFI\n"

newareah = false
newareah := top ? false : nz(newareah[1], false)
if negdivergence >= showlimit or hnegdivergence >= showlimit
    var line divlh = na
    var label labh = na
    if newareah                             // we remove old line until It reaches new pivot point (like animation ;)
        line.delete(divlh)
        label.delete(labh)
    newwd = not showhidden ?
       (not chwidth ? 2 :
       negdivergence <= 2 ? 2 :
       negdivergence <= 5 ? 3 :
       negdivergence <= 8 ? 4 : 5) :
       (not chwidth ? 2 :
       hnegdivergence <= 2 ? 2 :
       hnegdivergence <= 5 ? 3 :
       hnegdivergence <= 8 ? 4 : 5)
       
    divlh := line.new(bar_index - topc, high[topc], bar_index, high, color = color.red, width = newwd)
    if shownum or showindis
        txt = showindis ? showhidden ? hnegdivtxt : negdivtxt : ""
        txt := txt + (shownum ? showhidden ? tostring(hnegdivergence) : tostring(negdivergence) : "")
        labh := label.new(bar_index, na, text=txt, color= color.red, textcolor = color.white, style= label.style_labeldown, yloc=yloc.abovebar)
    newareah := true 

// Regular / Hidden positive Divergence
newbot = pivotlow(lb, 0) // check only left side
emptyl = true
if not na(newbot) and ((newbot < low[botc] and not showhidden) or   (newbot > low[botc] and showhidden))  // there must not close price lower than the line between last PL and current low
    diff = (newbot - low[botc]) / botc
    lline = newbot - diff                   // virtual line to check there is no close price lower than it
    for x = 1 to botc -1
        if close[x] < lline
            emptyl := false
            break
        lline := lline - diff
else
    emptyl := false

// check cut-through in indicators
nocut2(indi, len)=>
    _ret = true
    diff = (indi - nz(indi[len])) / len
    ln = indi - diff
    for x = 1 to len -1
        if nz(indi[x]) < ln
            _ret := false
            break
        ln := ln - diff
    _ret

rsiok := nocut2(rsi, botc)
macdok := nocut2(macd, botc)
deltamacdok := nocut2(deltamacd, botc)
momentok := nocut2(moment, botc)
cciok := nocut2(cci, botc)
obvok := nocut2(obv, botc)
stkok := nocut2(stk, botc)
dioscok := nocut2(diosc, botc)
vwmacdok := nocut2(vwmacd, botc)
cmfok := nocut2(cmf, botc)
mfiok := nocut2(Mfi, botc)

//positive regular divergence
posdivergence = 0
posdivtxt = ""
if emptyl and not na(newbot) and not showhidden
    if calcrsi and rsi[botc] < rsi and (not chcut or rsiok)
        posdivergence += 1
        posdivtxt := "RSI\n"
    if calcmacd and macd[botc] < macd  and (not chcut or macdok)
        posdivergence += 1
        posdivtxt := posdivtxt + "MACD\n"
    if calcmacda and deltamacd[botc] < deltamacd and (not chcut or deltamacdok)
        posdivergence += 1
        posdivtxt := posdivtxt + "MACD Hist\n"
    if calcmom and moment[botc] < moment and (not chcut or momentok)
        posdivergence += 1
        posdivtxt := posdivtxt + "Momentum\n"
    if calccci and cci[botc] < cci and (not chcut or cciok)
        posdivergence += 1
        posdivtxt := posdivtxt + "CCI\n"
    if calcobv and Obv[botc] < Obv and (not chcut or obvok)
        posdivergence += 1
        posdivtxt := posdivtxt + "OBV\n"
    if calcstoc and stk[botc] < stk and (not chcut or stkok)
        posdivergence += 1
        posdivtxt := posdivtxt + "Stoch\n"
    if calcdi and diosc[botc] < diosc and (not chcut or dioscok)
        posdivergence += 1
        posdivtxt := posdivtxt + "Diosc\n"
    if calcvwmacd and vwmacd[botc] < vwmacd and (not chcut or vwmacdok)
        posdivergence += 1
        posdivtxt := posdivtxt + "VWMacd\n"
    if calccmf and cmf[botc] < cmf and (not chcut or cmfok)
        posdivergence += 1
        posdivtxt := posdivtxt + "CMF\n"
    if calcmfi and Mfi[botc] < Mfi and (not chcut or mfiok)
        posdivergence += 1
        posdivtxt := posdivtxt + "MFI\n"

// Hidden Positive Divergences
hposdivergence = 0
hposdivtxt = ""
if emptyl and not na(newbot) and showhidden
    if calcrsi and rsi[botc] > rsi and (not chcut or rsiok)
        hposdivergence += 1
        hposdivtxt := "RSI\n"
    if calcmacd and macd[botc] > macd  and (not chcut or macdok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "MACD\n"
    if calcmacda and deltamacd[botc] > deltamacd and (not chcut or deltamacdok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "MACD Hist\n"
    if calcmom and moment[botc] > moment and (not chcut or momentok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "Momentum\n"
    if calccci and cci[botc] > cci and (not chcut or cciok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "CCI\n"
    if calcobv and Obv[botc] > Obv and (not chcut or obvok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "OBV\n"
    if calcstoc and stk[botc] > stk and (not chcut or stkok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "Stoch\n"
    if calcdi and diosc[botc] > diosc and (not chcut or dioscok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "Diosc\n"
    if calcvwmacd and vwmacd[botc] > vwmacd and (not chcut or vwmacdok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "VWMacd\n"
    if calccmf and cmf[botc] > cmf and (not chcut or cmfok)
        hposdivergence += 1
        hposdivtxt := hnegdivtxt + "CMF\n"
    if calcmfi and Mfi[botc] > Mfi and (not chcut or mfiok)
        hposdivergence += 1
        hposdivtxt := hposdivtxt + "MFI\n"

newareal = false
newareal := bot ? false : nz(newareal[1], false)
if posdivergence >= showlimit or hposdivergence >= showlimit
    var line divl = na
    var label lab = na
    if newareal                             // we remove old line until It reaches new pivot point (like animation ;)
        line.delete(divl)
        label.delete(lab)
    newwd = not showhidden ?
       (not chwidth ? 2 :
       posdivergence <= 2 ? 2 :
       posdivergence <= 5 ? 3 :
       posdivergence <= 8 ? 4 : 5) :
       (not chwidth ? 2 :
       hposdivergence <= 2 ? 2 :
       hposdivergence <= 5 ? 3 :
       hposdivergence <= 8 ? 4 : 5)
       
    divl := line.new(bar_index - botc, low[botc], bar_index, low, color = color.lime, width = newwd)
    if shownum or showindis
        txt = showindis ? showhidden ? hposdivtxt : posdivtxt : ""
        txt := txt + (shownum ? showhidden ? tostring(hposdivergence) : tostring(posdivergence) : "")
        lab := label.new(bar_index, na, text=txt, color= color.lime, textcolor = color.black, style = label.style_labelup, yloc=yloc.belowbar)
    newareal := true
 
alertcondition(posdivergence >= showlimit and not showhidden, title='Positive Divergence', message='Positive Divergence')
alertcondition(negdivergence >= showlimit and not showhidden, title='Negative Divergence', message='Negative Divergence')
alertcondition(hposdivergence >= showlimit and showhidden, title='Positive Hidden Divergence', message='Positive Hidden Divergence')
alertcondition(hnegdivergence >= showlimit and showhidden, title='Negative Hidden Divergence', message='Negative Hidden Divergence')
