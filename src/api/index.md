---
heroImage: ''
footer: Made by JayGreentree with ❤️
sidebar: auto
---
# Countdown

Create a new file in ``templates/_includes`` named ``countdown.twog`` insert the code below inside the file

```javascript
<script>
    $(document).ready(function() {
    $.ajax({
        url: "https://dcogb.ddev.site/api/countdown",
        success: function(result) {
            // if live is, well, true, then just show a link to it
            if (result.response.item.isLive == true) {
                showCountdownCompletion();
                // hide the 'next event' text, since the next event is NOW
                $(".lead-text").css("display", "none");
            } else {
                // otherwise parse the next start time out, and display the countdown
                var timeInMS = new Date(result.response.item.eventStartTime).getTime();
                startTimer(timeInMS);
                // show the 'next event' text, since the next event is NOW
                $(".lead-text").css("display", "block");
            }
        }
    })
});
</script>
<script>
    function showCountdownTimer() {
    $(".countdown-timer").css("display", "flex");
    $(".countdown-complete").css("display", "none");
}

function showCountdownCompletion() {
    $(".countdown-timer").css("display", "none");
    $(".countdown-complete").css("display", "flex");
}

function startTimer(countdownTargetDateMS) {
    x = setInterval(function() { tickTimer(countdownTargetDateMS); }, 1000);
}

// countdownTargetDateMS should be a full epoch date in milliseconds
function tickTimer(countdownTargetDateMS) {

    var nowDateMS = new Date().getTime();

    // get the delta from now to the target time (all in milliseconds)
    var deltaTime = countdownTargetDateMS - nowDateMS;

    // calc needed values to break out the time parts
    var dayInMilliseconds = (1000 * 60 * 60 * 24);
    var hourInMilliseconds = (1000 * 60 * 60);
    var minutesInMilliseconds = (1000 * 60);

    var days = Math.floor(deltaTime / dayInMilliseconds);
    var hours = Math.floor(deltaTime % dayInMilliseconds / hourInMilliseconds);
    var minutes = Math.floor(deltaTime % hourInMilliseconds / minutesInMilliseconds);
    var seconds = Math.floor(deltaTime % minutesInMilliseconds / 1000);

    // update the actual markup, and there're always two digits
    $(".days").text(addLeadingZero(days));
    $(".hours").text(addLeadingZero(hours));
    $(".minutes").text(addLeadingZero(minutes));
    $(".seconds").text(addLeadingZero(seconds));

    if ((days + hours + minutes + seconds) > 0) {
        showCountdownTimer();
    } else {
        showCountdownCompletion();

        clearInterval(x);
    }
}

function addLeadingZero(value) {
    if (value < 10) {
        return "0" + value
    };

    return value;
}
</script>
```
```html
<div class="ccv-live-countdown">
    <style type="text/css">
    /* Countdown timer */
    .show-timer,
    .show-live {
        width: 100%;
    }

    .countdown-row {
        min-height: 75px;
        margin: 20px 0 40px 0;
    }

    .countdown-timer {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .countdown-complete {
        dispay: none;
        display: flex;
        width: 100%;
    }

    .time {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
    }

    p.time-digit {
        font-size: 34px;
        margin: 15px 5px 0 0;
        line-height: .9
    }

    p.time-label {
        font-size: 12px;
    }

    .spacer {
        border-left: 1px solid gray;
        height: 44px;
    }
    </style>
    <div class="countdown-timer">
        <div class="spacer"></div>
        <div class="time">
            <span class="days">00</span>
            <span class="time-label">days</span>
        </div>
        <div class="spacer"></div>
        <div class="time">
            <span class="hours">00</span>
            <span class="time-label">hours</span>
        </div>
        <div class="spacer"></div>
        <div class="time">
            <span class="minutes">00</span>
            <span class="time-label">minutes</span>
        </div>
        <div class="spacer"></div>
        <div class="time">
            <span class="seconds">00</span>
            <span class="time-label">seconds</span>
        </div>
        <div class="spacer"></div>
    </div>
    <div class="countdown-complete">
        <a href="/live/" target="_blank" class="btn btn-primary">Live now</a>
    </div>
</div>
```

To Add the countdown to your remplates use the following code

```twig
 {{ include '_includes/countdown'}}
```

# Countdown API

## Building the API for automating the countdown

The API template code requires either [Craft Calendar](https://plugins.craftcms.com/calendar) or [Calendarize](https://plugins.craftcms.com/calendarize) 

### Using Craft Calendar

```twig
{% header "Content-Type: application/json; charset=utf-8" %}
{% header "Access-Control-Allow-Origin: *" %}

{% set currentEventId = craft.app.request.segment(3) %}
{% for event in craft.calendar.events({id: 'not ' ~ currentEventId,rangeStart: 'today',rangeEnd: '1 month',limit: 1}) %}
{
    "next_timestamp": {{ event.startDate.format("U") }},
    "next_title": "{{ event.title }}",
    "next_description": "{{ event.description }}"
}
{% endfor %}
```

### Using Calendarize

```twig
{% for event in craft.entries.section('events') %}
    {
    "next_timestamp": {{ event.eventTime|date("U") }},
    "next_title": "{{ event.title }}",
    "next_description": "{{ event.description }}"
}
{% endfor %}
```