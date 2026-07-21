//@version=6
indicator("Wave Engine V3 - Module 2 FIXED", overlay = true, max_labels_count = 500, max_lines_count = 500)

//====================================================
// INPUTS
//====================================================

showCandidates = input.bool(true, "Show Candidate Extremes")
showPivots     = input.bool(true, "Show Confirmed Pivots")
showFib        = input.bool(true, "Show Fibonacci")

minRetrace = input.float(20.0, "Minimum Retracement %", minval = 0.0, maxval = 100.0)

//====================================================
// STATES
//====================================================

const int STATE_UP = 1
const int STATE_DOWN = -1

var int state = STATE_UP

//====================================================
// CANDIDATES
//====================================================

var float candidateHigh = na
var int candidateHighBar = na
var float candidateHighCandleLow = na

var float candidateLow = na
var int candidateLowBar = na
var float candidateLowCandleHigh = na

//====================================================
// CONFIRMED PIVOTS
//====================================================

var float confirmedHigh = na
var int confirmedHighBar = na

var float confirmedLow = na
var int confirmedLowBar = na

//====================================================
// WAVE START
//====================================================

var float waveStartPrice = na
var int waveStartBar = na

//====================================================
// FIB
//====================================================

var line fib0 = na
var line fib20 = na
var line fib50 = na
var line fib100 = na

//====================================================
// INITIALIZATION
//====================================================

if barstate.isfirst

    candidateHigh := high
    candidateHighBar := bar_index
    candidateHighCandleLow := low

    waveStartPrice := low
    waveStartBar := bar_index


//====================================================
// UP STATE
//====================================================

if state == STATE_UP

    if na(candidateHigh) or high > candidateHigh

        candidateHigh := high
        candidateHighBar := bar_index
        candidateHighCandleLow := low


    highRange = candidateHigh - waveStartPrice

    retraceFromHigh =
         highRange > 0 ?
         (candidateHigh - low) / highRange * 100 :
         0


    highConfirmed =
         bar_index > candidateHighBar and
         close < candidateHighCandleLow


    if highConfirmed

        confirmedHigh := candidateHigh
        confirmedHighBar := candidateHighBar

        waveStartPrice := confirmedHigh
        waveStartBar := confirmedHighBar


        candidateLow := low
        candidateLowBar := bar_index
        candidateLowCandleHigh := high


        state := STATE_DOWN


        if showPivots

            label.new(
                 confirmedHighBar,
                 confirmedHigh,
                 "H",
                 style = label.style_label_down,
                 color = color.red,
                 textcolor = color.white)


//====================================================
// DOWN STATE
//====================================================

if state == STATE_DOWN


    if na(candidateLow) or low < candidateLow

        candidateLow := low
        candidateLowBar := bar_index
        candidateLowCandleHigh := high



    lowRange = waveStartPrice - candidateLow

    retraceFromLow =
         lowRange > 0 ?
         (high - candidateLow) / lowRange * 100 :
         0



    lowConfirmed =
         bar_index > candidateLowBar and
         close > candidateLowCandleHigh



    if lowConfirmed

        confirmedLow := candidateLow
        confirmedLowBar := candidateLowBar


        waveStartPrice := confirmedLow
        waveStartBar := confirmedLowBar


        candidateHigh := high
        candidateHighBar := bar_index
        candidateHighCandleLow := low


        state := STATE_UP


        if showPivots

            label.new(
                 confirmedLowBar,
                 confirmedLow,
                 "L",
                 style = label.style_label_up,
                 color = color.green,
                 textcolor = color.white)
                 //====================================================
// CANDIDATE HIGH DISPLAY
//====================================================

plotshape(
     showCandidates and state == STATE_UP and bar_index == candidateHighBar,
     title = "Candidate High",
     style = shape.circle,
     location = location.abovebar,
     color = color.orange,
     size = size.tiny)


//====================================================
// CANDIDATE LOW DISPLAY
//====================================================

plotshape(
     showCandidates and state == STATE_DOWN and bar_index == candidateLowBar,
     title = "Candidate Low",
     style = shape.circle,
     location = location.belowbar,
     color = color.aqua,
     size = size.tiny)


//====================================================
// FIBONACCI
//====================================================

if showFib and state == STATE_DOWN and not na(confirmedHigh)

    fibRange = confirmedHigh - candidateLow

    fib0Price = confirmedHigh
    fib20Price = confirmedHigh - fibRange * 0.20
    fib50Price = confirmedHigh - fibRange * 0.50
    fib100Price = candidateLow


    if na(fib0)

        fib0 := line.new(
             confirmedHighBar,
             fib0Price,
             bar_index,
             fib0Price,
             color = color.gray,
             width = 1)


        fib20 := line.new(
             confirmedHighBar,
             fib20Price,
             bar_index,
             fib20Price,
             color = color.orange,
             width = 1)


        fib50 := line.new(
             confirmedHighBar,
             fib50Price,
             bar_index,
             fib50Price,
             color = color.yellow,
             width = 1)


        fib100 := line.new(
             confirmedHighBar,
             fib100Price,
             bar_index,
             fib100Price,
             color = color.gray,
             width = 1)



    else

        line.set_xy1(
             fib0,
             confirmedHighBar,
             fib0Price)

        line.set_xy2(
             fib0,
             bar_index,
             fib0Price)



        line.set_xy1(
             fib20,
             confirmedHighBar,
             fib20Price)

        line.set_xy2(
             fib20,
             bar_index,
             fib20Price)



        line.set_xy1(
             fib50,
             confirmedHighBar,
             fib50Price)

        line.set_xy2(
             fib50,
             bar_index,
             fib50Price)



        line.set_xy1(
             fib100,
             confirmedHighBar,
             fib100Price)

        line.set_xy2(
             fib100,
             bar_index,
             fib100Price)



//====================================================
// RESET FIB WHEN NEW UP WAVE STARTS
//====================================================

if state == STATE_UP and not na(fib0)

    line.delete(fib0)
    line.delete(fib20)
    line.delete(fib50)
    line.delete(fib100)

    fib0 := na
    fib20 := na
    fib50 := na
    fib100 := na
