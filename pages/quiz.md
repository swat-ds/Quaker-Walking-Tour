---
title: Swarthmore Quaker Campus Quiz
layout: page
permalink: /quiz/
---

<div class="container my-5">
    <p class="lead">Type in the name of Swarthmore buildings. How many are named after Quakers?</p>
    
    <!-- Input Box -->
    <div class="input-group mb-3" style="max-width: 500px;">
        <input type="text" id="bldgInput" class="form-control" placeholder="Type a building name (e.g., Parrish)...">
        <button class="btn btn-primary" onclick="checkBuilding()">Submit</button>
    </div>

    <!-- Live Scoreboards -->
    <div class="row text-center my-4">
        <div class="col-md-4">
            <div class="card p-3 bg-light"><h3>Total Found: <span id="totalCount">0</span></h3></div>
        </div>
        <div class="col-md-4">
            <div class="card p-3 bg-success text-white"><h3>Quaker Named: <span id="quakerCount">0</span></h3></div>
        </div>
    </div>

    <!-- Feedback Message -->
    <div id="feedback" class="alert d-none" role="alert"></div>

    <!-- Discovered List -->
    <h4 class="mt-4">Buildings You've Found:</h4>
    <ul id="discoveredList" class="list-group"></ul>
</div>

<!-- Game Engine Logic -->
<script>
// This is using the csv file called quiz-info
const campusData = [
    {% for item in site.data.quiz-info %}
    {
        title: "{{ item.title | escape }}",
        // This strips away words like 'hall', 'house', 'library', 'memorial', and 'center' so players only have to type the main name for the point to count
        searchName: "{{ item.title | downcase | replace: ' hall', '' | replace: ' house', '' | replace: ' library', '' | replace: ' memorial', '' | replace: ' science and engineering library', '' | replace: ' science center', '' | replace: ' center', '' | strip }}",
        isQuaker: {% if item.is_quaker == 'yes' %}true{% else %}false{% endif %},
        note: "{{ item.description | escape }}"
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
];

const foundBuildings = new Set(); // This will hold the found buildings in a set
let quakerScore = 0; // This is the score for how many buildings they correctly guess as Quakers

function checkBuilding() {
    const inputField = document.getElementById("bldgInput"); // What the player inputs
    const guess = inputField.value.trim().toLowerCase(); // this takes the player's guess and makes it lower case
    const feedback = document.getElementById("feedback"); // This is the feedback given to the player
    
    feedback.className = "alert"; // Reset Bootstrap alert styling
    inputField.value = ""; // Empty the text input box for the next guess

    if (!guess) return;

    // Check if the player's guess matches 
    const match = campusData.find(b => b.searchName === guess || b.title.toLowerCase() === guess);

    // If the player's guess doesn't match
    if (!match) {
        feedback.classList.add("alert-danger");
        feedback.innerText = `"${guess}" wasn't found. Try again!`;
        feedback.classList.remove("d-none");
        return;
    }

    // if the player guesses the same thing twice
    if (foundBuildings.has(match.title)) {
        feedback.classList.add("alert-warning");
        feedback.innerText = `You already found ${match.title}!`;
        feedback.classList.remove("d-none");
        return;
    }

    // Process a new valid campus guess
    foundBuildings.add(match.title);
    if (match.isQuaker) {
        quakerScore++;
        feedback.classList.add("alert-success");
        feedback.innerText = `Correct! ${match.title} is named after a Quaker. (${match.note})`;
    } else {
        feedback.classList.add("alert-info");
        feedback.innerText = `So close! ${match.title} is a valid campus building, but it is not named after a Quaker. (${match.note})`;
    }
    feedback.classList.remove("d-none");

    updateScoreboard(match);
}

function updateScoreboard(match) {
    document.getElementById("totalCount").innerText = foundBuildings.size; // This is the total count of all buildings found
    document.getElementById("quakerCount").innerText = quakerScore; // This gets the correct number of quaker buildings found

    // This showcases the result for the player 
    const list = document.getElementById("discoveredList");
    const li = document.createElement("li");
    li.className = "list-group-item d-flex justify-content-between align-items-center mb-1";
    li.innerHTML = `<span><strong>${match.title}</strong> — ${match.note}</span>`;
    
    if (match.isQuaker) {
        li.innerHTML += `<span class="badge bg-success">Quaker Named</span>`;
    } else {
        li.innerHTML += `<span class="badge bg-secondary">Non-Quaker</span>`;
    }
    list.appendChild(li);
}

// Allow pressing 'Enter' in the input box to submit 
document.getElementById("bldgInput").addEventListener("keypress", function(e) {
    if (e.key === "Enter") checkBuilding();
});
</script>
