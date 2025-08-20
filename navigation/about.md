---
layout: post
title: About Me
permalink: /about/
comments: true
---

<style>
    /* Style looks pretty compact, trace grid-container and grid-item in the code */
    .grid-container {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); /* Dynamic columns */
        gap: 10px;
    }
    .grid-item {
        text-align: center;
    }
    .grid-item img {
        width: 100%;
        height: 100px; /* Fixed height for uniformity */
        object-fit: contain; /* Ensure the image fits within the fixed height */
    }
    .grid-item p {
        margin: 5px 0; /* Add some margin for spacing */
    }
</style>

<h4 style="color: white;">
I love playing and following both soccer and cricket! My favorite soccer team is Barcelona.
</h4>

<img src="{{site.baseurl}}/images/aboutme/sportsteams.jpg" height="150 px">

<footer class="site-footer">
</footer>

<!-- This grid_container class is for the CSS styling, the id is for JavaScript connection -->
<div class="grid-container" id="grid_container">
    <!-- content will be added here by JavaScript -->
</div>

<script> 
    // 1. Make a connection to the HTML container defined in the HTML div
    var container = document.getElementById("grid_container"); // This container connects to the HTML div

    // 2. Define a JavaScript object for our http source and our data rows for the Living in the World grid
    var http_source = "https://upload.wikimedia.org/wikipedia/commons/";
    var living_in_the_world = [
        {"flag": "0/01/Flag_of_California.svg", "greeting": "Hey", "description": "Born in California"},
        {"flag": "a/a4/Flag_of_the_United_States.svg", "greeting": "Hi", "description": "Will always live in the US"},
        {"flag": "4/41/Flag_of_India.svg", "greeting": "Namaste", "description": "Cultural ties to India"},
    ]; 
    
    // 3a. Consider how to update style count for size of container
    // The grid-template-columns has been defined as dynamic with auto-fill and minmax

    // 3b. Build grid items inside of our container for each row of data
    for (const location of living_in_the_world) {
        // Create a "div" with "class grid-item" for each row
        var gridItem = document.createElement("div");
        gridItem.className = "grid-item";  // This class name connects the gridItem to the CSS style elements
        // Add "img" HTML tag for the flag
        var img = document.createElement("img");
        img.src = http_source + location.flag; // concatenate the source and flag
        img.alt = location.flag + " Flag"; // add alt text for accessibility

        // Add "p" HTML tag for the description
        var description = document.createElement("p");
        description.textContent = location.description; // extract the description

        // Add "p" HTML tag for the greeting
        var greeting = document.createElement("p");
        greeting.textContent = location.greeting;  // extract the greeting

        // Append img and p HTML tags to the grid item DIV
        gridItem.appendChild(img);
        gridItem.appendChild(description);
        gridItem.appendChild(greeting);

        // Append the grid item DIV to the container DIV
        container.appendChild(gridItem);
    }
</script>

Although my ancestry lies in India, my parents, who are from different parts of India, both moved to the United States for college. They met each other at UTA, and eventually moved to San Diego.

<footer class="site-footer">
</footer>


<h4 style="color: white;">
One of my most major accomplishments has been captaining a team in middle school to win the CyberPatriot national finals.
</h4>

<img src="{{site.baseurl}}/images/aboutme/teamphoto.png" height="150 px">

<footer class="site-footer">
</footer>


<h4 style="color: white;">
Outside of sports, I have played both the piano and the cello for almost 5 years each.


<img src="{{site.baseurl}}/images/aboutme/piano&cello.jpg" height="150 px">

<footer class="site-footer">
</footer>

<h4 style="color: white;">
On a more fun side, I really enjoy listening to music and also playing video games! My favorite video game is Fortnite, which I play on my Xbox.
</h4>

<img src="{{site.baseurl}}/images/aboutme/fortniteandmusic.jpg" height="150 px">

<br>
<br>
<br>