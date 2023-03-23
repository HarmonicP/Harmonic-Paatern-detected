//@version=5
// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © reees
//@version=5

indicator("Harmonic Pattern Detection, Prediction, and Backtesting System","Harmonics",overlay=true,max_lines_count=500,max_labels_count=500,max_bars_back=500)


//----------------------------------------- 
// inputs and vars  
//-----------------------------------------

// Pattern Type Inputs
var bullOn = input.bool(true, "Bullish", inline="type_b", group="Include")
var bearOn = input.bool(true, "Bearish", inline="type_b", group="Include")
var incOn = input.bool(true, "Potential/Incomplete", inline="type_b", group="Include")
var hsOn = input.bool(true, "Only high scoring", inline="type_b", group="Include",tooltip="Only show patterns that meet your 'If score is above' entry requirement.")
var i_lb = input.bool(false,"Lookback bars",group="Include",inline="h")
var i_lbn = input.int(100,"",group="Include",inline="h")
var gartOn = input.bool(true, "Gartley:    ", inline="gart", group="Types & Targets")
var gart_t1 = input.string(".618 AD", "Target 1", inline="gart", group="Types & Targets", options=[".382 AD",".5 AD",".618 AD",".382 XA",".5 XA",".618 XA","1.272 XA","1.618 XA",".382 CD",".5 CD",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var gart_t2 = input.string("1.272 AD", " Target 2", inline="gart", group="Types & Targets", options=[".618 AD","1.272 AD","1.618 AD",".618 XA","1.272 XA","1.618 XA",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var batOn = input.bool(true, "Bat:          ", inline="bat", group="Types & Targets")
var bat_t1 = input.string(".618 AD", "Target 1", inline="bat", group="Types & Targets", options=[".382 AD",".5 AD",".618 AD",".382 XA",".5 XA",".618 XA","1.272 XA","1.618 XA",".382 CD",".5 CD",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var bat_t2 = input.string("1.272 AD", " Target 2", inline="bat", group="Types & Targets", options=[".618 AD","1.272 AD","1.618 AD",".618 XA","1.272 XA","1.618 XA",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var bflyOn = input.bool(true, "Butterfly:  ", inline="bfly", group="Types & Targets")
var bfly_t1 = input.string(".618 AD", "Target 1", inline="bfly", group="Types & Targets", options=[".382 AD",".5 AD",".618 AD",".382 XA",".5 XA",".618 XA","1.272 XA","1.618 XA",".382 CD",".5 CD",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var bfly_t2 = input.string("1.272 AD", " Target 2", inline="bfly", group="Types & Targets", options=[".618 AD","1.272 AD","1.618 AD",".618 XA","1.272 XA","1.618 XA",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var crabOn = input.bool(true, "Crab:        ", inline="crab", group="Types & Targets")
var crab_t1 = input.string(".618 AD", "Target 1", inline="crab", group="Types & Targets", options=[".382 AD",".5 AD",".618 AD",".382 XA",".5 XA",".618 XA","1.272 XA","1.618 XA",".382 CD",".5 CD",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var crab_t2 = input.string("1.618 AD", " Target 2", inline="crab", group="Types & Targets", options=[".618 AD","1.272 AD","1.618 AD",".618 XA","1.272 XA","1.618 XA",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var sharkOn = input.bool(true, "Shark:       ", inline="shark", group="Types & Targets")
var shark_t1 = input.string(".382 AD", "Target 1", inline="shark", group="Types & Targets", options=[".382 AD",".5 AD",".618 AD",".382 XA",".5 XA",".618 XA","1.272 XA","1.618 XA",".382 CD",".5 CD",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var shark_t2 = input.string("C", " Target 2", inline="shark", group="Types & Targets", options=[".618 AD","1.272 AD","1.618 AD",".618 XA","1.272 XA","1.618 XA",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var cyphOn = input.bool(true, "Cypher:     ", inline="cyph", group="Types & Targets")
var cyph_t1 = input.string(".618 CD", "Target 1", inline="cyph", group="Types & Targets", options=[".382 AD",".5 AD",".618 AD",".382 XA",".5 XA",".618 XA","1.272 XA","1.618 XA",".382 CD",".5 CD",".618 CD","1.272 CD","1.618 CD","A","B","C"])
var cyph_t2 = input.string("1.618 XA", " Target 2", inline="cyph", group="Types & Targets", options=[".618 AD","1.272 AD","1.618 AD",".618 XA","1.272 XA","1.618 XA",".618 CD","1.272 CD","1.618 CD","A","B","C"])
// Entry/Stop Inputs
var e_afterC = input.bool(true,"Enter after Point C",group="Entry/Stop")
var e_lvlc = input.string("Nearest confluent PRZ level","       Enter at",options=["Nearest confluent PRZ level","Farthest confluent PRZ level","Between the two confluent PRZ levels","Nearest PRZ level","Farthest PRZ level"],group="Entry/Stop")
var e_aboveC = input.float(90,"       If score is above",group="Entry/Stop",tooltip="A trade will only be entered if the pattern's score is above the specified value. Note that when entering a trade after Point C, we have an incomplete score because we can't yet measure Point D's confluence with the PRZ. Therefore the final pattern score may differ from the incomplete score at the time of entry. Set this to 0 if you wish to enter a trade on every pattern.")
var e_afterD = input.bool(true,"Enter after Point D",group="Entry/Stop")
var e_lvldPct = input.float(1.0,"       Enter at limit % away from D",minval=0.0,step=.1,group="Entry/Stop",tooltip="Enter the trade after a valid point D has been confirmed, up to the limit of this % away from Point D. E.g. for a bullish pattern (long entry), if this value is set to 5%, an entry will be placed at the best possible price up to 5% above point D. If 'Entry after Point C' is also set, the entry will be at whichever level is reached first. \n\nIf the entry level is not reached, the pattern will not be included in the Success Rate and Return % statistics.")
var e_aboveD = input.float(90,"       If score is above",group="Entry/Stop",tooltip="A trade will only be entered if the pattern's score is above the specified value. Set this to 0 if you wish to enter a trade on every pattern.")
var e_tLimit = input.float(.5,"       Entry window (time limit)",group="Entry/Stop",tooltip="Time limit for order entry, specified in pattern lengths (e.g. '0.5' means half the total pattern length). If the time limit expires before the order is filled, it will be cancelled and no trade will be entered for the pattern.")
var stopPct = input.float(75,"Stop",step=1.0,minval=0.0,group="Entry/Stop",inline="stop")
var stopB = input.string("% of distance to target 1, beyond entry","",options=["% beyond X or D","% beyond Farthest PRZ level","% beyond Point D","% beyond entry","% of distance to target 1, beyond entry"],group="Entry/Stop",inline="stop",tooltip="Set stop-loss % beyond the specified level. If price reaches this level before the first target is hit, or before the target timeout period expires, the pattern will be considered a failure.\n\n'% beyond X or D' = percentage below Point X or Point D, whichever is farther from entry\n\n'% of distance to target 1' = a percentage of the distance from the entry level to target 1. \n\n'% beyond entry' = percentage above/below the entry level. \n\n'% beyond Point D' = percentage above/below Point D. \n\n'beyond Farthest PRZ level' = percentage above/below the Farthest PRZ level")
// Pattern Inputs
var t_b = input.int(1,"Pattern validation length (# trailing bars)",minval=1,group="Pattern",tooltip="The number of bars after pivot point D (or point C for incomplete patterns) before a pattern is considered valid. This affects how soon patterns will be drawn and entries can be placed.")
var pctErr = input.float(15.0,"Allowed fib ratio error %",step=1.0,minval=0.0,maxval=50.0,group="Pattern",inline="err")
var pctAsym = input.float(250.0,"Allowed leg length asymmetry %",step=1.0,minval=0.0,maxval=1000.0,group="Pattern",inline="asym",tooltip="A leg is considered valid if its length (ΔX/number of bars) is within this % of the average length of the other legs in the pattern.")
var w_e = input.float(4.0,"Weight",step=.1,minval=0.0,group="Pattern",inline="err",tooltip="A leg is considered valid if its retracement (ΔY) ratio is within this % of the defined harmonic ratio. Weight determines the weight of retracement % error in the total score calculation for a pattern.")
//var w_a = input.float(0.0,"Weight",step=.1,minval=0.0,group="Pattern",inline="asym",tooltip="A leg is considered valid if its length (ΔX/number of bars) is within this % of the average length of the other legs in the pattern. Weight determines the weight of length asymmetry in the total score calculation for a pattern.")
var tLimitMult = input.float(3,"Pattern time limit",group="Pattern",step=.1,minval=.1,tooltip="Time limit for a completed pattern to reach the projected targets. Value is specified in terms of total pattern length (point X to point D), i.e. a value of 1 will allow one pattern length to elapse before the pattern times out and can no longer be considered successful. Patterns that time out will not count towards the success rates in the results table.")
var w_p = input.float(2.0,"Weight of PRZ level confluence",step=.1,minval=0.0,group="Pattern",tooltip="Weight applied to Potential Reversal Zone fib level confluence in the total score calculation for a pattern. The closer together the two closest PRZ fib levels are, the higher the score.")
var w_d = input.float(3.0,"Weight of point D / PRZ level confluence",step=.1,minval=0.0,group="Pattern",tooltip="Weight applied to the confluence of point D with the Potential Reversal Zone levels in the total score calculation for a pattern. The closer point D is to either of the two confluent PRZ fib levels, the higher the score. ")

// Alert Inputs
//var a_on = input.bool(true, "Alert", inline="alert", group="Alerts")
var a_type = input.string("Both", "Alert for", options=["Potential patterns","Complete patterns","Both"], inline="alert", group="Alerts")

// Display Inputs
var c_bline = input.color(color.new(color.green,20), "Bullish lines", group="Display")
var c_beline = input.color(color.new(color.red,20), "Bearish lines", group="Display")
var c_blab = input.color(color.new(color.green,75), "Bullish labels", group="Display")
var c_belab = input.color(color.new(color.red,75), "Bearish labels", group="Display")
var l_txt = input.color(color.new(color.white,20), "Label text", group="Display")

var int[] includeTps = array.new_int()
if barstate.isfirst
    if gartOn
        array.push(includeTps,1)
    if batOn
        array.push(includeTps,2)
    if bflyOn
        array.push(includeTps,3)
    if crabOn
        array.push(includeTps,4)
    if sharkOn
        array.push(includeTps,5)
    if cyphOn
        array.push(includeTps,6)
var h.harmonic_params params = h.init_params(pctErr,pctAsym,includeTps,w_e,w_p,w_d)     // scoring and validation parameters for xabcd_harmonic objects

// xabcd_harmonic object pointers
var h.xabcd_harmonic[] bullGart = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bullBat = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bullBfly = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bullCrab = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bullShark = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bullCyph = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bearGart = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bearBat = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bearBfly = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bearCrab = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bearShark = array.new<h.xabcd_harmonic>()
var h.xabcd_harmonic[] bearCyph = array.new<h.xabcd_harmonic>()
// var int[] lastX = array.new_int(5,0)

// temp/last/incomplete pattern structures
var h.xabcd_harmonic[] pending = array.new<h.xabcd_harmonic>()
var label[] fullIpL = array.new_label(0)
var line[] fullIpLn = array.new_line(0)
var linefill[] fullIpLf = array.new_linefill(0)
var h.xabcd_harmonic[] inc = array.new<h.xabcd_harmonic>()
var int[] inc_lastX = array.new_int(4,0)

// Stat totals
var int[] pTot = array.new_int(0)
var int[] tTot = array.new_int(0)
var float[] t1Tot = array.new_float(0)
var float[] t2Tot = array.new_float(0)
var float[] arTot = array.new_float(0)
var float[] trTot = array.new_float(0)

//-----------------------------------------
// functions
//-----------------------------------------

// Type of harmonic pat
// Assumes pattern is complete, and therefore only one can be true
tp(h1,h2,h3,h4,h5,h6) =>
    switch
        h1 => 1
        h2 => 2
        h3 => 3
        h4 => 4
        h5 => 5
        => 6

nToArray(n) =>
    switch n
        "a1" => bullGart
        "a2" => bullBat
        "a3" => bullBfly
        "a4" => bullCrab
        "a5" => bullShark
        "a6" => bullCyph
        "b1" => bearGart
        "b2" => bearBat
        "b3" => bearBfly
        "b4" => bearCrab
        "b5" => bearShark
        "b6" => bearCyph

typeToArray(t,tp) =>
    n = (t ? "a" : "b") + str.tostring(tp)
    nToArray(n)

t1(tp) =>
    switch tp
        6 => cyph_t1
        5 => shark_t1
        4 => crab_t1
        3 => bfly_t1
        2 => bat_t1
        => gart_t1

t2(tp) =>
    switch tp
        6 => cyph_t2
        5 => shark_t2
        4 => crab_t2
        3 => bfly_t2
        2 => bat_t2
        => gart_t2

// get target
targets(tp) =>
    [t1(tp),t2(tp)]

harmonic_xabcd_targets(tp,xY,aY,bY,cY,dY) =>
    tgt1 = t1(tp)
    tgt2 = t2(tp)
    [t1,t2,_] = t.harmonic_xabcd_targets(xY,aY,bY,cY,dY,tgt1,tgt2)
    [t1,t2]

// Timeout period
tLimit(xX,dX) =>
    int((dX - xX)*tLimitMult)

incTLimit(xX,cX) =>
    avg = (cX-xX)/3
    int(avg * (1 + pctAsym/100))    // time out after max possible bars based on asymmetry parameter

// Entry has timed out
eTimeout(xX,dX) =>
    bar_index - dX > int((dX - xX)*e_tLimit)

// Pattern still active within timeout period
stillActive(xX,dX) =>
    bar_index - tLimit(xX,dX) <= dX and eTimeout(xX,dX) == false

// make sure score is high enough to enter a trade
noEntry(h.xabcd_harmonic p) =>
    not na(p.d.x) ? p.score < (e_aboveD/100) : p.score < (e_aboveC/100)

entry(p) =>
    if noEntry(p)
        [na,na,na]
    else
        t.harmonic_xabcd_entry(p.bull,p.tp,p.x.y,p.a.y,p.b.y,p.c.y,p.d.y,e_afterC,e_lvlc,e_afterD,e_lvldPct)

// Determine if entry level was reached
entryHit(p) =>
    if p.eHit or p.eHit == false
        [p.eHit,p.e.x,p.e.y]
    else if (not na(p.d.x) and eTimeout(p.x.x,p.d.x)) or noEntry(p)
        [false,na,na]
    else
        [_,afterC,afterD] = entry(p)
        [eH,eX,eY] = t.xabcd_entryHit(p.bull, afterC, afterD, p.d.x, e_afterC, e_afterD, t_b)
        if not na(eY) and ((p.bull and p.t1<eY) or (p.bull==false and p.t1>eY))
            [na,na,na]
        else
            [eH==false?na:eH,eX,eY]

// Determine if pattern has succeeded or failed, or neither (na)
success(p) =>
    // if stop or target 2 already hit, nothing to check
    if p.sHit or p.t2Hit
        [p.t1Hit,p.t2Hit,p.sHit,na,na,na,na]
    // if within time limit and trade active / entry hit, check targets/stop
    else if bar_index <= (p.d.x + tLimit(p.x.x,p.d.x)) and p.eHit
        t.tradeClosed(p.e.x,p.e.y,p.stop,p.t1Hit,p.t2Hit,p.t1,p.t2)
    // else nothing to update
    else
        [p.t1Hit,p.t2Hit,p.sHit,na,na,na,na]

alertMsg(p) =>
    if na(p.d.x)
        "Potential " + h.get_name(p) + " is forming."
    else
        h.get_name(p) + " has formed."

deleteFip() =>
    for lbl in fullIpL
        label.delete(lbl)
    array.clear(fullIpL)
    //
    for ln in fullIpLn
        line.delete(ln)
    array.clear(fullIpLn)
    //
    for lf in fullIpLf
        linefill.delete(lf)
    array.clear(fullIpLf)

removePending(pid) =>
    if array.size(pending) > 0
        for i=0 to array.size(pending)-1
            p = array.get(pending,i)
            if p.pid == pid
                if i == array.size(pending)-1
                    deleteFip()
                array.remove(pending,i)
                break

successTxt(p) =>
    if p.t2Hit
        " (Success - Target 1, Target 2)"
    else if p.t1Hit
        " (Success - Target 1)"
    else if p.sHit
        " (Failed)"
    else if noEntry(p) and stillActive(p.x.x,p.d.x)
        " (No entry)"
    else if (p.eHit==false or na(p.eHit)) and stillActive(p.x.x,p.d.x) == false
        " (Missed entry)"
    else if na(p.eHit) and stillActive(p.x.x,p.d.x)
        " (Entry pending)"
    else if na(p.t1Hit) and stillActive(p.x.x,p.d.x)
        " (Targets pending)"
    else
        " (Timed out)"

ratToStr(r) =>
    na(r) ? "NA    " : str.tostring(r, "0.000")

reToStr(r) =>
    na(r) ? "NA    " : str.tostring(r, "0.0") + "%"

// Pattern tooltip
ttTxt(p) =>
    [rb,rc,rd1,rd2] = t.harmonic_xabcd_fibDispTxt(p.tp)
    [_,_,e] = entry(p)
    l1 = h.get_name(p) + successTxt(p) + "\n\n"
    l2 = (p.invalid_d?"Incomplete":"Total") + " Score:  " + str.tostring(p.score*100,"#.###") + "\n"
    l3 = "   Leg retracement accuracy:  " + str.tostring((1-p.score_eAvg)*100, "#.##") + "%\n"
    l42 = "   PRZ level confluence:  " + (p.tp==6 ? "NA" : (str.tostring(p.score_prz*100, "#.##") + "%")) + "\n"
    l43 = "   Point D confluence with PRZ:  " + (p.invalid_d?"NA (D unconfirmed)":str.tostring((1-p.score_eD)*100, "#.##") + "%") +"\n"
    l5 = "\n              Actual     % Err       Theoretical\n"
    l6 = "AB/XA     " + str.tostring(p.r_xb, "0.000") + "      " + (p.tp==5 ? "NA     " : (str.tostring(p.re_xb*100, "00.0")+"%")) + "      " + rb + "\n"
    l7 = "BC/AB     " + str.tostring(p.r_ac, "0.000") + "      " + str.tostring(p.re_ac*100, "00.0") + "%      " + rc + "\n"
    l8 = "CD/BC     " + ratToStr(p.r_bd) + "      " + (p.tp==6 ? "NA    " : reToStr(p.re_bd*100)) + "       " + rd1 + "\n"
    l9 = (p.tp==6 ? "CD/XC     " : "AD/XA     ") + ratToStr(p.r_xd) + "      " + reToStr(p.re_xd*100) + "       " + rd2 + "\n"
    l91 = "\nTarget 1:  " + str.tostring(p.t1,"#.#####") + "\nTarget 2:  " + str.tostring(p.t2,"#.#####")
    l92 = "\nEntry:  " + str.tostring(na(p.e.y)?e:p.e.y,"#.#####")
    l93 = na(p.stop) ? "" : "\nStop:  " + str.tostring(p.stop,"#.#####")
    l1 + l2 + l3 + l42 + l43 + l5 + l6 + l7 + l8 + l9 + l92 + l93 + l91

status(p) =>
    if p.t2Hit
        " ✅✅"
    else if p.t1Hit
        " ✅"
    else if p.sHit
        " ❌"
    else if stillActive(p.x.x,p.d.x) == false and (p.eHit == false or na(p.eHit))
        " ⛔"
    else if stillActive(p.x.x,p.d.x) and noEntry(p)
        " ⛔"
    else if stillActive(p.x.x,p.d.x)
        " ⏳"
    else
        " 🕝"
        
lbTxt(p,status) =>
    t.harmonic_xabcd_symbol(p.tp) + " " + str.tostring(math.round(p.score,3)*100) + status

incLbTxt(p) =>
    "Potential " + t.harmonic_xabcd_symbol(p.tp) + " (" + str.tostring(p.score*100,"#.##") + ")"

erasePattern(p) =>
    h.erase_pattern(p)
    h.erase_label(p)

deleteInc(string pid) =>
    n = array.size(inc)
    if not na(pid) and n > 0
        for j=0 to n-1
            p = array.get(inc,j)
            if pid == p.pid
                array.remove(inc,j)
                erasePattern(p)
                break

drawPattern(p) =>
    if incOn or not na(p.d.x)
        status = status(p)
        lbTxt = not na(p.d.x) ? lbTxt(p,status) : incLbTxt(p)
        [_,e,_] = entry(p)
        tt = not na(p.d.x) ? ttTxt(p) : draw.incTtTxt(p.tp,h.get_name(p),p.r_xb,p.re_xb,p.r_ac,p.re_ac,p.prz_bN,p.prz_bF,p.prz_xN,p.prz_xF,p.score,e)
        h.draw_pattern(p, p.bull?c_bline:c_beline)
        if p.invalid_d
            line.set_style(array.get(p.pLines,3),line.style_dashed)
        h.draw_label(p, p.bull?c_blab:c_belab, l_txt, lbTxt, tt)

lowest(n, o=0) =>
    if n >= o
        v = low[o]
        for i=o to n
            if low[i] < v
                v := low[i]
        v

highest(n, o=0) =>
    if n >= o
        v = high[o]
        for i=o to n
            if high[i] > v
                v := high[i]
        v

notLast(p) =>   // check if not same pattern as last completed pattern of same type
    comp = typeToArray(p.bull,p.tp)
    h.xabcd_harmonic last = array.size(comp) == 0 ? na : array.get(comp,array.size(comp)-1)
    na(last) ? true : last.x.x!=p.x.x or last.a.x!=p.a.x or last.b.x!=p.b.x

addIncompletePattern(t,h1,h2,h3,h4,h5,h6,xX,xY,aX,aY,bX,bY,cX,cY) =>
    btps = array.from(h1,h2,h3,h4,h5,h6)
    tps = u.boolToIntArr(btps)
    lowest = lowest(bar_index-cX)
    highest = highest(bar_index-cX)
    int dX = na
    float dY = na
    // check if pattern already exists (incomplete or pending)
    exists = false
    iN = array.size(inc)
    iP = array.size(pending)
    if iN > 0
        for i=0 to iN-1
            p = array.get(inc,iN-1-i)       // more likely to find it at top of stack
            if p.x.x==xX and p.a.x==aX and p.b.x==bX
                if p.c.x == cX
                    exists := true
                else
                    // if new point C, delete the old inc pattern in favor of this one
                    array.remove(inc,iN-1-i)
                    erasePattern(p)
                break
    if exists==false and iP > 0
        for i=0 to iP-1
            p = array.get(pending,iP-1-i)
            if p.x.x==xX and p.a.x==aX and p.b.x==bX and p.c.x==cX
                exists := true
                break
    
    if exists == false
        // add separate incomplete pattern for each potential harmonic type
        for tpe in tps
            tp = tpe+1
            if exists == false
                pat = h.init(xX,xY,aX,aY,bX,bY,cX,cY,dX,dY,params,tp)
                if not na(pat)
                    [_,eC,_] = entry(pat)
                    if na(eC) or (not na(eC) and ((t and eC < lowest) or (t==false and eC > highest)))
                        if notLast(pat)
                            array.push(inc,pat)
                            if incOn and (not na(eC) or not e_afterC)
                                drawPattern(pat)
                            if a_type == "Potential patterns" or a_type == "Both"
                                alert(alertMsg(pat),alert.freq_once_per_bar_close) // temporarily changing to fire on bar close until real-time bar multiple alert issue is resolved

setTargets(p) =>
    [t1,t2] = targets(p.tp)
    h.set_target(p,1,calc_target=t1)
    h.set_target(p,2,calc_target=t2)

incPid(p) =>
	str.tostring(p.tp) + "_"
     + str.tostring(p.x.x) + "_"
     + str.tostring(p.a.x) + "_"
     + str.tostring(p.b.x) + "_"
     + str.tostring(p.c.x) + "_"
     + str.tostring(na)

// Draw completed pattern and update data structures
addCompleted(p,force=false) =>
    if hsOn==false or noEntry(p)==false
        a = typeToArray(p.bull,p.tp)   
        array.push(a,p)  // add pattern
        // check if we need to delete incomplete pattern, only if not already adding from an incomplete pattern and allowing entry after D
        // (otherwise keep incomplete pattern to continue checking for entry after C)
        if force == false
            if e_afterD
                deleteInc(incPid(p))
            p.invalid_d := false
        setTargets(p)
        drawPattern(p)
        [upper,lower] = t.harmonic_xabcd_przRange(p.prz_bN,p.prz_bF,p.prz_xN,p.prz_xF)
        p.stop := t.harmonic_xabcd_stop(stopB,stopPct,p.bull,p.x.y,p.d.y,upper,lower,p.t1,p.e.y)
        if noEntry(p)==false
            array.push(pending,p)
        // fire alert if appropriate
        if a_type == "Complete patterns" or a_type == "Both"
            alert(alertMsg(p),alert.freq_once_per_bar_close) // temporarily changing to fire on bar close until real-time bar multiple alert issue is resolved

// Add pattern to completed pattern structures
addValidPattern(p,force=false) =>
    bool lasteH = na
    float lastScore = na
    h.xabcd_harmonic pat = na
    a = typeToArray(p.bull,p.tp)
    if array.size(a) > 0
        // check last pattern of same type
        last = array.get(a,array.size(a)-1)
        lastScore := last.score
        // if A, B or C is different = new pattern
        if p.a.x!=last.a.x or p.b.x!=last.b.x or p.c.x!=last.c.x
            addCompleted(p,force)
            pat := p        
        // if ABC are same but D is beyond last pattern's D, replace it with this one. We want to draw the
        // new/updated pattern and calculate its updated score, but maintain any entry/targets that have
        // already been hit.
        else if (p.bull and p.d.y < last.d.y) or (p.bull==false and p.d.y > last.d.y)
            if (last.score <= p.score) or force or last.invalid_d // update if new pattern has a higher score, last pattern had invalid D, or forcing from entry After Point C
                if last.eHit                                       // IF last pattern entry was already hit, use the last pattern
                    lasteH := true
                    if force==false
                        last.invalid_d := false
                    h.init(last.x.x,last.x.y,last.a.x,last.a.y,last.b.x,last.b.y,last.c.x,last.c.y,p.d.x,p.d.y,params,last.tp,last)
                    [upper,lower] = t.harmonic_xabcd_przRange(last.prz_bN,last.prz_bF,last.prz_xN,last.prz_xF)
                    last.stop := t.harmonic_xabcd_stop(stopB,stopPct,last.bull,last.x.y,last.d.y,upper,lower,last.t1,last.e.y)
                    drawPattern(last)                   // redraw with new D
                    setTargets(last)                    // reset targets for new D
                    pat := last
                else                                // ELSE, replace last pattern with the new pattern
                    removePending(last.pid)
                    erasePattern(array.pop(a))
                    addCompleted(p,force)
                    pat := p
    else
        addCompleted(p,force)
        pat := p

    // Update newly added pattern
    if not na(pat)
        // update entry, if necessary
        if lasteH
            draw.eHitLbl(pat.e.x,pat.e.y,pat.d.x,pat.d.y,p.bull,true)
        else
            [eHit,eX,eY] = entryHit(p)
            if eHit
                pat.eHit := true
                pat.e.x := eX
                pat.e.y := eY
                draw.eHitLbl(eX,eY,pat.d.x,pat.d.y,pat.bull)
                h.draw_label(p, p.bull?c_blab:c_belab, l_txt, lbTxt(p,status(p)), ttTxt(p))
        ""
        
updatePendingPatterns() =>
    h.xabcd_harmonic[] new = array.new<h.xabcd_harmonic>()
    if array.size(pending) > 0
        for i=0 to array.size(pending)-1
            ip = array.get(pending,i)
            [eH,eHx,eHy] = entryHit(ip)
            tLimit = tLimit(ip.x.x,ip.d.x)
            expired = bar_index == (ip.d.x + tLimit + 1) or eH==false
            [t1h,t2h,sH,t1x,t1y,t2x,t2y] = success(ip)
            // if time has expired or there's nothing left to update, no longer pending
            if not (expired or (eH and (not na(t2h) or t1h == false or sH)))
                array.push(new,ip)
            // if anything to update, update completed array entry and label if necessary            
            if expired 
             or (eH
                 and ((not na(t1h) and na(ip.t1Hit)) or (not na(t2h) and na(ip.t2Hit)))) 
             or (eH != ip.eHit) or (eH and na(ip.eHit)) or (eH==false and na(ip.eHit)) 
             or sH
                if (eH or eH==false) and na(ip.eHit)
                    ip.eHit := eH
                    ip.e.x := eHx
                    ip.e.y := eHy            
                    draw.eHitLbl(eHx,eHy,ip.d.x,ip.d.y,ip.bull)
                // targets will be na if eH==false, so no need to also check eH 
                if t1h and na(ip.t1Hit)
                    ip.t1Hit := true
                    draw.tHitLbl(t1x,t1y,eHx,eHy,ip.bull)
                else if t1h == false
                    ip.t1Hit := false
                    draw.sHitLbl(t1x,t1y,eHx,eHy,ip.bull)   // only draw stop X if no target was already hit
                if sH
                    ip.sHit := true
                    //draw.sHitLbl(bar_index,ip.stop,eHx,eHy,ip.bull)
                if t2h
                    ip.t2Hit := true
                    draw.tHitLbl(t2x,t2y,eHx,eHy,ip.bull)
                h.draw_label(ip, ip.bull?c_blab:c_belab, l_txt, lbTxt(ip,status(ip)), ttTxt(ip))
    new

updateIncompletePatterns() =>
    h.xabcd_harmonic[] new = array.new<h.xabcd_harmonic>()
    if array.size(inc) > 0
        for p in inc
            [upper,lower] = t.harmonic_xabcd_przRange(p.prz_bN,p.prz_bF,p.prz_xN,p.prz_xF)
            tLimit = incTLimit(p.x.x,p.c.x)
            [eH,eHx,eHy] = entryHit(p)
            if eH and e_afterC
                if notLast(p)
                    p.d.x := eHx
                    p.d.y := eHy
                    p.invalid_d := true
                    erasePattern(p)
                    addValidPattern(p,true)
                else
                    erasePattern(p)
            // Don't keep incomplete pattern if it's timed out or has been invalidated
            else if bar_index == (p.c.x + tLimit + 1)
                     or (p.bull and (high > p.c.y or low < lower))
                     or (p.bull==false and (low < p.c.y or high > upper))
                erasePattern(p)
            else
                array.push(new,p)
    new

drawFullInProgress() =>
    if array.size(pending) > 0
        last = array.get(pending,array.size(pending)-1)
        if not na(last.d.x)
            bb = last_bar_index - last.d.x
            tLimit = tLimit(last.x.x,last.d.x)
                                                                            // Only draw if...
            if bb <= tLimit                                                 // within pattern time limit and
               and noEntry(last)==false                                     // entry is possible and
               and (eTimeout(last.x.x,last.d.x)==false or last.eHit)        // entry has not timed out and
               //and (na(last.t1Hit) or (last.t1Hit and na(last.t2Hit)))
               and not last.t2Hit                                           // targets remain to be hit and
               and not last.sHit                                            // stop has not been hit
                deleteFip()                 // delete previously drawn completed pattern in progress
                [highest,lowest] = t.harmonic_xabcd_przRange(last.prz_bN,last.prz_bF,last.prz_xN,last.prz_xF)
                [bcNt,_] = harmonic_xabcd_targets(last.tp,last.x.y,last.a.y,last.b.y,last.c.y,last.prz_bN)
                [bcFt,_] = harmonic_xabcd_targets(last.tp,last.x.y,last.a.y,last.b.y,last.c.y,last.prz_bF)
                [xaNt,_] = harmonic_xabcd_targets(last.tp,last.x.y,last.a.y,last.b.y,last.c.y,last.prz_xN)
                [xaFt,_] = harmonic_xabcd_targets(last.tp,last.x.y,last.a.y,last.b.y,last.c.y,last.prz_xF)
                stop = last.stop
                [e,_,_] = entry(last)
                entry = na(last.e.y) ? e : last.e.y
                [ln,lb,lf] = draw.xabcd_inProgress(last.bull,last.tp,tLimit>500?500:tLimit,entry,stop,last.t1,last.t2,bcNt,bcFt,xaNt,xaFt,
                                                   last.x.x,last.x.y,last.a.y,last.b.x,last.b.y,last.c.y,last.d.x,last.d.y,c_bline,c_beline,l_txt)
                for l in ln
                    array.push(fullIpLn,l)
                for l in lb
                    array.push(fullIpL,l)
                for l in lf
                    array.push(fullIpLf,l)
            else
                deleteFip()
    else
        deleteFip()

rowValues(tp) =>
    a1 = typeToArray(true,tp)
    a2 = typeToArray(false,tp)
    float[] ra = array.new_float(0)
    t1_tot = 0
    t2_tot = 0
    closed = 0
    if array.size(a1) > 0
        for p in a1
            if (not na(p.t1Hit) or p.sHit) and p.eHit
                if p.t2Hit
                    r = (p.t2/p.e.y) - 1
                    array.push(ra,r)
                    t2_tot+=1
                    t1_tot+=1
                    closed+=1
                else if p.t1Hit
                    r = (p.t1/p.e.y) - 1
                    array.push(ra,r)
                    t1_tot+=1
                    closed+=1
                else if p.sHit
                    r = (p.stop/p.e.y) - 1
                    array.push(ra,r)
                    closed+=1
    //for p in a2
    if array.size(a2) > 0
        for p in a2
            if (not na(p.t1Hit) or p.sHit) and p.eHit
                if p.t2Hit
                    r = (p.e.y/p.t2) - 1
                    array.push(ra,r)
                    t2_tot+=1
                    t1_tot+=1
                    closed+=1
                else if p.t1Hit
                    r = (p.e.y/p.t1) - 1
                    array.push(ra,r)
                    t1_tot+=1
                    closed+=1
                else if p.sHit
                    r = (p.e.y/p.stop) - 1
                    array.push(ra,r)
                    closed+=1
    tot = array.size(a1) + array.size(a2)
    array.push(pTot,tot)
    array.push(tTot,closed)
    if closed > 0
        array.push(t1Tot,(t1_tot/closed)*100)
        array.push(t2Tot,(t2_tot/closed)*100)
    if not na(array.avg(ra))
        array.push(arTot,array.avg(ra)*100)
    if not na(array.sum(ra))
        array.push(trTot,array.sum(ra)*100)
    st1 = closed>0 ? str.tostring((t1_tot/closed)*100,"#.##")+"%" : "NA"
    st2 = closed>0 ? str.tostring((t2_tot/closed)*100,"#.##")+"%" : "NA"
    ravg = not na(array.avg(ra)) ? str.tostring(array.avg(ra)*100,"#.##")+"%" : "NA"
    rtot = not na(array.sum(ra)) ? str.tostring(array.sum(ra)*100,"#.##")+"%" : "NA"

    [str.tostring(tot),str.tostring(closed),st1,st2,ravg,rtot]

printStats() =>
    if barstate.islast
        nR = array.size(includeTps) + 3
        r = 0
        t = table.new(position.bottom_left, 7, nR, bgcolor = color.new(color.black,30), border_width = 1)
        table.cell(t, 0, 0, " ", text_color=color.white, text_halign=text.align_center)
        table.cell(t, 1, 0, "Patterns", text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 2, 0, "Trades", text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 3, 0, "T1 Success", text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 4, 0, "T2 Success", text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 5, 0, "Avg Return %", text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 6, 0, "Total Return %", text_color=color.white, text_halign=text.align_center,text_size=size.small)        
        for tp in includeTps
            r+=1
            [tot,trd,st1,st2,ar,tr] = rowValues(tp)
            table.cell(t, 0, r, "   "+t.harmonic_xabcd_symbol(tp)+"   ", text_color=color.white, text_halign=text.align_center,text_size=size.small)
            table.cell(t, 1, r, tot, text_color=color.white, text_halign=text.align_center,text_size=size.small)
            table.cell(t, 2, r, trd, text_color=color.white, text_halign=text.align_center,text_size=size.small)
            table.cell(t, 3, r, st1, text_color=color.white, text_halign=text.align_center,text_size=size.small)
            table.cell(t, 4, r, st2, text_color=color.white, text_halign=text.align_center,text_size=size.small)
            table.cell(t, 5, r, ar, text_color=color.white, text_halign=text.align_center,text_size=size.small)
            table.cell(t, 6, r, tr, text_color=color.white, text_halign=text.align_center,text_size=size.small)

        r+=1
        t1Total = not na(array.avg(t1Tot)) ? (str.tostring(array.avg(t1Tot),"#.##") + "%") : "NA"
        t2Total = not na(array.avg(t2Tot)) ? (str.tostring(array.avg(t2Tot),"#.##") + "%") : "NA"
        arTotal = not na(array.avg(arTot)) ? (str.tostring(array.avg(arTot),"#.##") + "%") : "NA"
        trTotal = not na(array.sum(trTot)) ? (str.tostring(array.sum(trTot),"#.##") + "%") : "NA"
        table.cell(t, 0, r, "Total", text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 1, r, str.tostring(array.sum(pTot)), text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 2, r, str.tostring(array.sum(tTot)), text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 3, r, t1Total, text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 4, r, t2Total, text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 5, r, arTotal, text_color=color.white, text_halign=text.align_center,text_size=size.small)
        table.cell(t, 6, r, trTotal, text_color=color.white, text_halign=text.align_center,text_size=size.small)
        r+=1
        keyTxt = "✅ = Success | ❌ = Failure | 🕝 = Timed out* | ⛔ = No entry* | ⏳ = In progress* \n*Not included in Success Rate/Return % statistics"
        table.cell(t, 0, r, keyTxt, text_color=color.white, text_halign=text.align_right, text_size=size.small)
        table.merge_cells(t,0,r,6,r)
        array.clear(pTot),array.clear(tTot),array.clear(t1Tot),array.clear(t2Tot),array.clear(arTot),array.clear(trTot)

validD(p,dX,dY) =>
    // If CD has valid length symmetry...
    if dX <= (incTLimit(p.x.x,p.c.x) + p.c.x)
        if t.pat_xabcd_testSym(p.a.x-p.x.x, p.b.x-p.a.x, p.c.x-p.b.x, p.d.x-p.c.x, pctAsym)
            highest = highest(bar_index-p.c.x,1)
            lowest = lowest(bar_index-p.c.x,1)
            // If no intermediate high/low invalidates the CD leg...
            if ((p.bull and dY <= lowest) or (p.bull==false and dY >= highest)) 
             and ((p.bull and p.c.y >= highest) or (p.bull==false and p.c.y <= lowest))
                // validate CD retracement for this pattern type
                xa = math.abs(p.x.y - p.a.y)
                bc = math.abs(p.b.y - p.c.y)
                cd = math.abs(p.c.y - dY)
                ad = math.abs(p.a.y - dY)
                xc = math.abs(p.x.y - p.c.y)
                tp = p.tp
                p_types = switch tp
                    1 => array.from(true,false,false,false,false,false)
                    2 => array.from(false,true,false,false,false,false)
                    3 => array.from(false,false,true,false,false,false)
                    4 => array.from(false,false,false,true,false,false)
                    5 => array.from(false,false,false,false,true,false)
                    6 => array.from(false,false,false,false,false,true)
                [t1,t2,t3,t4,t5,t6] = t.test_cd(cd,bc,xa,xc,ad,pctErr,p_types)
                t1 or t2 or t3 or t4 or t5 or t6
            else
                false
        else
            false
    else
        false

// Check for a valid pivot point D based on the pattern confirmation length parameter (i.e. checking bar_index[t_b])
checkForValidD() =>
    x = bar_index[t_b]          // bar of interest
    bool isLow = true
    bool isHigh = true
    l = low
    h = high
    
    // Validate pivot
    for i=0 to t_b-1            // check bars after potential pivot point
        if l[i] < l[t_b]
            isLow := false
            break    
    if isLow                    // check bars before potential pivot point
        for i=t_b+1 to t_b+3        // need at least 3 bars before 
            if l[i] < l[t_b]
                isLow := false
                break

    for i=0 to t_b-1
        if h[i] > h[t_b]
            isHigh := false
            break
    if isHigh
        for i=t_b+1 to t_b+3
            if h[i] > h[t_b]
                isHigh := false
                break
    
    // Validate pattern (CD leg)
    if isLow or isHigh
        h.xabcd_harmonic[] new = array.new<h.xabcd_harmonic>()
        // Check incomplete patterns
        for p in inc
            if isLow and p.bull
                if validD(p, x, l[t_b])
                    array.push(new, h.init(p.x.x, p.x.y, p.a.x, p.a.y, p.b.x, p.b.y, p.c.x, p.c.y, x, l[t_b], params, p.tp))
            if isHigh and p.bull==false
                if validD(p, x, h[t_b])
                    array.push(new, h.init(p.x.x, p.x.y, p.a.x, p.a.y, p.b.x, p.b.y, p.c.x, p.c.y, x, h[t_b], params, p.tp))
        // Check completed pending patterns
        for p2 in pending
            if isLow and p2.bull
                if validD(p2, x, l[t_b])
                    array.push(new, h.init(p2.x.x, p2.x.y, p2.a.x, p2.a.y, p2.b.x, p2.b.y, p2.c.x, p2.c.y, x, l[t_b], params, p2.tp))
            if isHigh and p2.bull==false
                if validD(p2, x, h[t_b])
                    array.push(new, h.init(p2.x.x, p2.x.y, p2.a.x, p2.a.y, p2.b.x, p2.b.y, p2.c.x, p2.c.y, x, h[t_b], params, p2.tp))

        for n in new
            if not na(n)
                addValidPattern(n)

// find new XABC (potential) pattern
find_pattern(pl,t=true) =>
    if ((t and bullOn) or (t==false and bearOn))
        [f,xX,xY,aX,aY,bX,bY,cX,cY] = t.pat_xabcdIncomplete(t,pl)
        if f and (i_lb==false or bar_index >= last_bar_index-i_lbn)
            [h,h1,h2,h3,h4,h5,h6] = t.harmonic_xabcd_validateIncomplete(xX,xY,aX,aY,bX,bY,cX,cY,pctErr,pctAsym,gartOn,batOn,bflyOn,crabOn,sharkOn,cyphOn)
            if h
                addIncompletePattern(t,h1,h2,h3,h4,h5,h6,xX,xY,aX,aY,bX,bY,cX,cY)
    
// Find XABCD patterns of various pivot lengths
find(bull=true) =>
    // Could probably do this more efficiently with recursion, but Pine prohibits it. Loops are all
    // top-tested though, so it's probably not as expensive as it looks.
    find_pattern(3,bull)
    find_pattern(4,bull)
    find_pattern(5,bull)
    find_pattern(6,bull)
    find_pattern(7,bull)
    find_pattern(8,bull)
    find_pattern(9,bull)
    find_pattern(10,bull)
    find_pattern(11,bull)
    find_pattern(12,bull)
    find_pattern(13,bull)
    find_pattern(14,bull)
    find_pattern(15,bull)
    find_pattern(16,bull)
    find_pattern(17,bull)
    find_pattern(18,bull)
    find_pattern(19,bull)
    find_pattern(20,bull)
    
//-----------------------------------------
// Main
//-----------------------------------------

// update patterns in progress
inc := updateIncompletePatterns()       // update potential/incomplete pattern
pending := updatePendingPatterns()      // update any completed patterns pending results

// find new patterns
find()                      // find bullish patterns
find(false)                 // find bearish patterns

checkForValidD()

// last bar business
if barstate.islast
    drawFullInProgress()        // draw most recent complete pattern, if still in progress
    printStats()                // compile stats and draw results table
