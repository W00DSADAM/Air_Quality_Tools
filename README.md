Everything we've covered building the NowCast, AQI and Unit Conversion tools and combining them into one site.
How the site fits together
The site is one GitHub repo, Air_Quality_Tools, published by GitHub Pages at w00dsadam.github.io/Air_Quality_Tools/. Every page is a plain text file the browser reads top to bottom.
Air_Quality_Tools/
  index.html     the menu page (opens first)
  nowcast.html   NowCast calculator
  aqi.html       AQI calculator
  units.html     Unit conversion calculator
  style.css      shared rules: menu cards, top bar, focus glow, button hover
The three layers of a page
• HTML is what's on the page: headings, boxes, buttons.
• CSS is how it looks: colors, spacing, layout. It lives in a <style> block in the <head>, or in a separate file like style.css.
• JavaScript is what it does: the math, building rows, reacting to typing. It lives in a <script> block near the end of <body>.
Working with GitHub
• Commit = save a version. Every commit is kept in the history, so an old version can always be recovered. There's no limit on commits.
• Pages republishes the site after each commit. Watch Actions: an orange dot means publishing, and a green check means it's live. Pages has a soft limit of about 10 builds per hour.
• Habit: fix everything you see, then commit once.
• Addresses are case-sensitive. aqi.html and AQI.html are different files online, even though Windows treats them as the same. Name web files in lowercase with hyphens.
• Relative links like href="aqi.html" mean "the file next to this one," so they work online and on your own computer.
Seeing your changes
• Ctrl + F5 reloads the page and skips the browser's saved copy.
• Browsers hold onto CSS files longer than pages. If a style change doesn't show up, open the CSS file directly and press Ctrl + F5, or change the link to style.css?v=2. Bump the number each time you edit it.
• Test locally first: save the code as a .html file and double-click it. There's no publishing wait.
• F12 opens the developer tools: Console shows JavaScript errors and lets you run code; Elements shows the page's HTML and, in the Styles pane, every CSS rule that applies to whatever you click.
The AQI rules built into the code
Each pollutant gets its own AQI from its own breakpoint table, and the overall AQI is the highest of them. These are the EPA rules (40 CFR Part 58 Appendix G, 2024 update) the AQI page follows.
The equation
Find the table row whose concentration range holds the reading, then:
AQI = \frac{AQI_{hi} - AQI_{lo}}{C_{hi} - C_{lo}} \times (C - C_{lo}) + AQI_{lo}
Round the result to a whole number. In the code, each row is [C_lo, C_hi, AQI_lo, AQI_hi] and the function formula(c, r) does the math.
Truncate first
Cut the reading to EPA's decimal places before looking it up. Don't round it. This keeps edge readings out of the gaps between rows, like 9.0 to 9.1.
Pollutant
Units
Keep
Example
PM2.5
µg/m³
1 decimal
35.47 becomes 35.4, so AQI 100, not 101
PM10
µg/m³
whole number
54.9 becomes 54
Ozone
ppb
whole number
54.9 becomes 54, so AQI 50, not 51
SO2
ppb
whole number
75.6 becomes 75
Tables that stop early
Table
Covers
Outside its range
Ozone 8-hr
AQI 0–300 (0–200 ppb)
Above 200 ppb, use 1-hour ozone
Ozone 1-hr
AQI 101–500 (125+ ppb)
Below 125 ppb, the 8-hour value decides
SO2 1-hr
AQI 0–200 (0–304 ppb)
Above 304 ppb, use 24-hour SO2
SO2 24-hr
AQI 201–500 (305+ ppb)
Below 305 ppb, the 1-hour value decides
PM2.5 and PM10 run all the way to 500. Past the top row, the Hazardous row's formula keeps going, so the AQI can go above 500. For example, PM2.5 400 µg/m³ gives AQI 649.
Other rules
• Negative readings count as 0.
• 8-hour ozone average: needs at least 6 of the 8 hours. Divide by the number of hours you have, then truncate to a whole ppb.
• Categories depend on the AQI, not the pollutant: Good 0–50, Moderate 51–100, Unhealthy for Sensitive Groups 101–150, Unhealthy 151–200, Very Unhealthy 201–300, Hazardous 301+.
• NowCast weights recent hours more heavily when air quality is changing fast: weight factor = 1 − (max − min) ÷ max, kept between 0.5 and 1, with at least 2 of the 3 newest hours required.
HTML
HTML is made of elements: an opening tag, the content, and a closing tag with a slash, like <h1>AQI Calculator</h1>. Every opening tag needs its closing partner, or everything after it ends up inside it.
Page skeleton
Element
What it does
<!DOCTYPE html>
First line of every page. Tells the browser it's modern HTML
<html lang="en">
Wraps the whole page. lang names the language
<head>
Settings the visitor doesn't see: title, fonts, CSS
<meta charset="utf-8">
Lets the page use symbols like µ, ³ and ° safely
<meta name="viewport" ...>
Makes the page fit a phone screen instead of shrinking
<title>
The text on the browser tab
<link rel="stylesheet" href="style.css">
Loads CSS from a separate file
<style>
CSS written inside the page
<body>
Everything the visitor sees
<script>
JavaScript. Placed at the end of <body>, so the elements it uses already exist
Content elements
Element
What it does
Where we used it
<h1>, <h2>
Page title, section titles
Page and card titles
<p>
A paragraph (one line or block of text)
Each pollutant row
<div>
A plain box for grouping things
Cards, the scale, the inputs area
<span>
A plain box inside a line of text
The small AQI badges, the result
<input>
A box you type in
Every reading
<label>
Text that names an input box
"PM2.5 (µg/m³):"
<button>
A clickable button
Use current hour, Average
<a href="...">
A link
Menu cards, top bar
<nav>
A group of site links
The top bar
Attributes
Attributes go inside the opening tag and add information: name="value".
• id is a unique name for one element, so JavaScript can find it: id="result". Only one element per page can have a given id.
• class is a style name that many elements can share: class="card". Separate two classes with a space: class="small-aqi moderate".
• href is where a link goes. type, min, max, value set up an input box, like the "Last hour" box: type="number" min="0" max="23" value="20".
• onclick, onchange, oninput name JavaScript to run when something happens: a click, a finished change, or every keystroke.
• aria-current="page" marks the link to the page you're on. Screen readers announce it, and our CSS colors it blue.
Habits
• Indent what's inside an element one level more than the element itself. It shows at a glance what's inside what.
• Copy whole blocks with the code block's Copy button. A one-line paste of <nav class="topnav"> without its </nav> pulled the whole page into the top bar.
CSS
A CSS rule is a selector (what to style) followed by properties in braces:
.card {
  background: #1b1b1b;
  border-radius: 10px;
}
Each property line is name: value;. The colon is followed by a space, and the semicolon separates one setting from the next. The semicolon on the last line before } is optional, but always add it.
Selectors
Selector
Styles
Example
input
Every element of that type
All input boxes
.card
Every element with that class
All three cards
#result
The one element with that id
The big AQI badge
.card h2
An h2 inside a .card (space = inside)
Card titles
h1, h2
Both (comma = "and also")
Blue titles on NowCast
button:hover
While the mouse is over it
Buttons lighten
input:focus
While it's selected for typing
Blue glow
button:active
While it's being clicked
Button presses down
a[aria-current="page"]
Elements with that attribute
Current page in the top bar
In a selector, a colon touches the words on both sides: button:hover, never button: hover.
Sizes and spacing
• padding is space inside an element's edge. margin is space outside it, between it and its neighbors.
• Two values mean top/bottom, then left/right: padding: 16px 20px.
• margin: 40px auto centers a box with a max-width: the browser splits the leftover space evenly on both sides.
• max-width lets a box shrink on small screens but stops it growing past a limit.
• Units: px is pixels, % is a share of the parent's width. max(5vw, 16px) means 5% of the window's width, but never less than 16px.
Color
• Hex colors like #1b1b1b are three pairs: red, green, blue, from 00 to ff. Higher means lighter. #000000 is black and #ffffff is white.
• rgba(108, 180, 238, 0.25) is red, green, blue, plus alpha, how see-through it is. 0.25 is 25% strength.
• On a black page, a dark shadow is invisible. Cards stand out by being slightly lighter, with a thin border.
Borders, corners, shadows
• border: 1px solid #2e2e2e: thickness, style (solid, dashed, dotted), color.
• border-radius rounds the corners.
• box-shadow: 0 2px 8px color: sideways shift, downward shift, blur, color. With a fourth size, 0 0 0 3px, it spreads into a solid ring: that's the focus glow.
• Triangles from borders: a box with width: 0; height: 0, a colored top border, and transparent left and right borders shows only a downward triangle. That's the scale marker.
Layout
• display: inline-block with a width gives every label the same width, so the input boxes line up.
• display: flex lines up everything inside an element in a row: the top bar links and the color scale segments. gap sets the space between them.
• display: grid with grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)) fits as many columns as there's room for, each at least 280px. 1fr means one share of the leftover space. That's the Units page.
• position: relative on a parent and position: absolute on a child let you place the child exactly, using top and left. That's how the marker and tick numbers sit on the scale.
• overflow: hidden cuts off anything that sticks out. It gives the scale its rounded ends, and it's why the marker had to move outside the scale.
• transform: translateX(-50%) shifts an element left by half its own width, so its center lands on the spot.
• white-space: nowrap keeps text on one line.
Motion
• transition: left 0.4s animates a change instead of jumping: the marker slides. List several with commas: transition: border-color 0.2s, box-shadow 0.2s.
Variables
NowCast defines colors once on :root as variables, like --bg: #000000, and uses them everywhere as var(--bg). Change one line, and the whole page follows. light-dark(A, B) picks A in light mode and B in dark mode. color-scheme: dark makes the page always dark.
Which rule wins
1.  Specificity first. The more specific selector wins: an id beats a class, a class beats an element name, and input:focus beats plain input.
2.  Order breaks ties. When two rules are equally specific, the one lower down wins. That's why <link> to style.css goes above each page's <style>: the page gets the final say on ties.
3.  Only matching properties compete. Two rules can both style input. Where they set different properties, both apply.
CSS never shows errors. A typo (a missing semicolon, 7pm instead of 7px, o.2s instead of 0.2s) just makes the browser skip that line or rule.
JavaScript
JavaScript runs top to bottom when the page loads, then waits for clicks and typing. Every statement ends with a semicolon, and // starts a comment the computer ignores.
Variables
• var total = 0; creates a variable named total holding 0. A variable has to be created with var before you read it or add to it. Otherwise the Console shows "total is not defined."
• Changing it: total = total + c; adds c to whatever total already holds. This "running total" pattern adds up the 8 hours.
• Text needs quotes, numbers don't: "none", "block" and "50%" are text. Without quotes, JavaScript looks for a variable by that name.
Lists (arrays)
• [0, 9.0, 0, 50] is a list. Items are counted from 0, so r[0] is the first item and r[3] is the fourth.
• Lists of lists are tables: pm25[1] is the second row, and pm25[1][0] is that row's first number.
• table.length is how many items there are. The last one is table[table.length - 1].
• stops.push(50) adds an item to the end.
• A list can hold tables. In pollutants, pm25 without quotes is the actual table, while "pm25" with quotes is just text used as a name tag.
• Put what's different in the list, not the code. Each pollutant's label, units, table, message and decimal places live in its row, so adding a pollutant means adding one row. The loop doesn't change.
Functions
function formula(c, r) {
  return Math.round((r[3] - r[2]) / (r[1] - r[0]) * (c - r[0]) + r[2]);
}
• A function is a named set of steps. c and r are parameters: the values you hand it when you call it, like formula(11.1, pm25[1]).
• return hands back an answer and stops the function right there.
• null means "no answer." aqiFor returns it when a reading is outside a table's range.
• One reusable function (aqiFor) works with any table, so six pollutants share one piece of math.
Loops
for (var i = 0; i < pollutants.length; i++) {
  // runs once for each pollutant
}
• Start at 0, keep going while i is less than the length, and add 1 (i++) each time.
• Counting down: for (var i = 7; i >= 0; i--) builds the 8 hour boxes oldest first.
• continue skips the rest of this pass and moves to the next item. We use it for blank boxes and typos.
• for (var other in mbarPer) goes through every name in an object. That's how the pressure converter updates every other unit.
Decisions
• if (...) { } else { } runs one block or the other. An if can sit inside another if.
• Comparisons: == equal, != not equal, < <= > >=. One = sets a value, and two compare.
• || means "or": if (t == "" || p == "") means "if either box is empty."
Numbers and text
Code
What it does
Number("12a")
Converts text to a number. Gives NaN for anything that isn't a clean number
parseFloat("12a")
Also converts, but quietly gives 12. That's why we switched to Number
isNaN(c)
Asks "is this Not a Number?"
text.trim()
Removes spaces from the start and end
Math.round(x)
Rounds to the nearest whole number
Math.floor(x)
Rounds down. Used for truncating
Math.max(c, 0)
The bigger of the two. Turns negatives into 0
Math.min(x, 100)
The smaller of the two. Keeps the marker on the scale
Math.pow(10, 1)
10 to the power of 1
x % 24
The remainder after dividing by 24. Wraps hours past midnight
"in-" + "pm25"
Joins text: "in-pm25"
Truncating to 1 decimal: multiply by 10, round down, divide by 10: 35.47 → 354.7 → 354 → 35.4. The tiny + 0.000001 protects against computers storing 35.4 as 35.39999.
Working with the page
• document.getElementById("result") finds an element by its id.
• .value is what's typed in an input box. .textContent is the text shown in an element.
• .className = "small-aqi moderate" sets its classes, which changes its look.
• .style.left = "50%" and .style.display = "none" set one CSS property directly. "none" hides an element, and "block" shows it.
• Building elements: document.createElement("input") makes a new one, you set its id and text, then parent.appendChild(child) puts it on the page. Loops build the pollutant rows, the hour boxes, the scale segments and the ticks this way.
• box.innerHTML = "" empties an element, so it can be rebuilt.
• Building ids the same way twice links code together: the loop gives a box the id "in-" + p[0], and calculateAll rebuilds that same id to find it again.
Events
• onclick, onchange and oninput run code when someone clicks, finishes a change, or types.
• box.oninput = calculateAll; has no parentheses. It hands over the function to run later. calculateAll() with parentheses would run it once, right now.
• Changing .value from code doesn't fire oninput. That's why average8 calls calculateAll() itself after filling in the ozone box.
• new Date() reads the computer's clock, and .getHours() gives the hour from 0 to 23.
Debugging
Most bugs we hit were one wrong character. The skill is knowing where each kind shows up: JavaScript usually complains in the Console, while HTML and CSS fail silently.
Bugs we actually hit
Bug
What you saw
Lesson
hightest instead of highest
"highest is not defined"
A misspelled variable is a different variable
Extra } in calculateAll
"Unexpected token '}'" a few lines later
Errors show up after the real mistake. Check that every { has a partner
textConcent instead of textContent
No error. The big number stayed after clearing boxes
A misspelled property quietly makes a new, useless one
Missing var c = Number(text);
"c is not defined"
Create a variable before you use it
continute instead of continue
Error only when typing a non-number
Code that only runs in unusual cases needs its own test
Missing var bad = 0;
Average button stopped working
Same lesson: create it first
marker.style.left = none;
"none is not defined"
CSS values in JavaScript need quotes
Missing ; after width: 100px
White, wide input boxes, invisible white text
CSS read two lines as one invalid line and dropped both
7pm instead of 7px
Lopsided triangle
CSS skips a line with an unknown unit
button: hover, :ative, o.2s
Hover and press effects didn't work
Selector colons touch both words. Letter o isn't zero
Only <nav class="topnav"> pasted
Title and cards squished in a row
A missing closing tag swallows the rest of the page
Files named AQIcalculator.html
Links would give 404
Names and links must match exactly, including capitals
Old style.css in the browser
A correct change didn't show
Caching. Ctrl + F5, or style.css?v=2
A routine that works
1.  Read the Console (F12). It names the error and the line. Look a few lines above that line, too.
2.  Nothing in the Console? Then suspect HTML or CSS. Use Elements to click the broken thing and check the Styles pane: a rule that's missing or crossed out points to a typo above it.
3.  Test one piece by itself. Type aqiFor(11.1, pm25) in the Console to check a function without the page.
4.  Test with a table of known answers, including the odd cases: blank boxes, negatives, typos like 12a, values past the end of a table.
5.  Line up your indentation. When braces or tags are off, the indents stop lining up.
6.  Only run Console code you understand. Chrome blocks pasting there until you type "allow pasting" because scammers trick people into pasting harmful code.
Self-check
Try each question before reading its answer.
1.  What's the difference between id and class?
    ◦ An id names exactly one element, so JavaScript can find it. A class is a style name that many elements can share.
2.  Why does <link rel="stylesheet" href="style.css"> go above each page's <style>?
    ◦ When two rules are equally specific, the later one wins. Loading style.css first lets each page's own CSS have the final say.
3.  style.css comes first, yet input:focus in it still beats input in aqi.html. Why?
    ◦ Specificity is checked before order. input:focus is more specific than input.
4.  What does .card h2 style, and how is that different from .card, h2?
    ◦ .card h2 styles h2s inside a card. .card, h2 styles every card and every h2.
5.  What's the difference between padding and margin?
    ◦ Padding is space inside an element's edge. Margin is space outside it.
6.  Why did the scale marker have to move out of #scale?
    ◦ The scale has overflow: hidden for its rounded ends, which would cut off the triangle sitting above the bar.
7.  What does box.oninput = calculateAll; do, and what would go wrong with calculateAll()?
    ◦ It tells the box to run calculateAll every time someone types. With parentheses, it would run once while the page loads and never again.
8.  Why does average8 still call calculateAll() itself?
    ◦ Setting .value from code doesn't fire oninput. Only typing does.
9.  Why use Number(text) instead of parseFloat(text)?
    ◦ parseFloat("12a") quietly gives 12. Number("12a") gives NaN, so the typo can be caught.
10.  PM2.5 is 35.47 µg/m³. What AQI should the calculator show, and why?
    ◦ 100 (Moderate). It truncates to 35.4 first, the top of the Moderate row. Rounding to 35.5 would give 101.
11.  Ozone (1-hr) is 100 ppb. What does the page show?
    ◦ "Use 8-hr." The 1-hour table starts at 125 ppb, so aqiFor returns null and the page shows that row's message.
12.  PM2.5 is 11.1, PM10 is 100 and Ozone (8-hr) is 65. What's the overall AQI?
    ◦ 84 (Moderate), from ozone. The individual AQIs are 55, 73 and 84, and the highest one sets the overall AQI.
13.  Your CSS change doesn't show up, and the Console has no errors. What do you check?
    ◦ First, a typo just above the rule: a missing ; or }, a bad unit, or a space in a selector. Then caching: Ctrl + F5, or open style.css directly. Check that Actions shows a green check.
14.  A link works when you double-click the file on your computer but gives a 404 online. Likely cause?
    ◦ A capital-letter mismatch. Windows ignores case, and GitHub Pages doesn't.
15.  How would you add a new pollutant, like NO2, to the AQI page?
    ◦ Add its breakpoint table, then add one row to the pollutants list (id, label, units, table, message, decimals). The loop builds the box, and the overall AQI picks it up automatically.
