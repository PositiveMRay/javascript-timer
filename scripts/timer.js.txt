// AlarmClock.js - Turn on a periodic beep at a set time.
// Updated 11.1.19

var targetTime;

// context and oscillator should be objects so changes to them
// within functions are not lost
var context = {};
var oscillator = {};

var timerOn = false;

// timer is used to start and stop the periodic beep
var beepTimer = null;
var sleepTimer = null;

let beepEnabled = true;
const BEEP_INTERVAL = 1500;
const BEEP_DURATION = 200;

function getTargetTime() {
    return document.getElementById("targetTime").value;
}

function timeToDate(timeString) {
    let date = new Date();
    date.setHours(timeString.substr(0,2));
    date.setMinutes(timeString.substr(3,2));
    date.setSeconds(0);
    return date;
}

function padWithZeroes(text, paddingZeroes) {
    if (text.length > paddingZeroes) {
        paddingZeroes = text.length;
    }
    let paddedText = "";
    // fill paddedText with zeroes up to point where the
    // text passed into the function is to be added
    for (var i = 0; i < paddingZeroes - text.length; i++) {
        paddedText += "0";
    }

    // overlay end of string with text
    return paddedText.concat(text);
}

function startTimer() {
// Find out when the user wants to start the beep
    // let timerDiv = getTimerDiv();
    document.getElementById("message").innerHTML = prompt("Your Message, please:");
    targetTime  = timeToDate(getTargetTime());
    sleepTimer = setInterval(testTargetTime, 500);
    timerOn = true;
}

function buildCountDownText() {
    if (timerOn !== true) {
	return "00:00:00";
    }
    let currentTime = new Date();
    let targetTime = timeToDate(document.getElementById("targetTime").value);
    targetTime.setHours(targetTime.getHours() - currentTime.getHours());
    targetTime.setMinutes(targetTime.getMinutes() - currentTime.getMinutes());
    targetTime.setSeconds(targetTime.getSeconds() - currentTime.getSeconds());
    let countDownText = "";
    countDownText = padWithZeroes(targetTime.getHours().toString(),2) + ":";
    countDownText += padWithZeroes(targetTime.getMinutes().toString(),2) + ":";
    countDownText += padWithZeroes(targetTime.getSeconds().toString(),2);
    return countDownText;
}

// testTargetTime() - test to see if target time has passed.
// If not, print countdown to page.
function testTargetTime() {
    // Get today's date and the current time into a variable named now.
    let now = new Date();
    // if target time has passed, never mind printing the count down to the page.
    if (targetTime < now) {
        // might not need to stop sleepTimer
	// clearInterval(sleepTimer);
        return;
    }
    let countDownText = document.getElementById("countDown");
    countDownText.innerText = buildCountDownText();
    if (now.getHours() === targetTime.getHours() &&
        now.getMinutes() === targetTime.getMinutes() &&
        now.getSeconds() === targetTime.getSeconds()) {
        // stop the countdown
	// - will call from startBeeping. clearInterval(sleepTimer);
	// trigger the alarm
        startBeeping();
    }
}

// Beep functions follow:

// setupOscillator() - we use an oscillator emitting an audible sine wave as the beep.
function setupOscillator() {
    context = new AudioContext();
    oscillator = context.createOscillator();
    oscillator.type = "sine";
    oscillator.connect(context.destination);
}

function beep() {
    setupOscillator()
    oscillator.start();
    setTimeout(function () {
        oscillator.stop();
    }, BEEP_DURATION);
}

function startBeeping() {
	console.log("startBeeping function triggered");
    clearInterval(sleepTimer);
    if (beepTimer !== undefined || beepTimer !== null) { // has beepTimer been set?
        clearInterval(beepTimer)
    }
    beepTimer = setInterval(beep, BEEP_INTERVAL);
}

function stopBeeping() {
    if (sleepTimer !== null) {
        clearInterval(sleepTimer);
    }

    if (beepTimer) {
        beepEnabled = false;
        clearInterval(beepTimer);
	beepTimer = null;
    }

    timerOn = false;
    oscillator.stop();
    document.getElementById('message').innerText = "Timer not in use";
}
