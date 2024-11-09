<script>
    import { onMount } from "svelte";

    let characters = [];
    let wordBoundaries = []; // Stores the index where each word starts
    let nextLetter = "";
    let key = "";
    let charIndex = 0;
    let selection = "prose";
    export let lessonContent = "Content"

    async function resetWords() {
        if (selection == "prose") {
            const res = {
               author: "Reid",
            };
            let rawWords = lessonContent.split(" ");
            characters = [];
            wordBoundaries = [0];

            // Insert words and spaces as separate "words"
            rawWords.forEach((word, i) => {
                // Add each character of the word
                for (let char of word) {
                    characters.push(char);
                }
                // Mark the start of next word
                wordBoundaries.push(characters.length);

                // Add space as its own word (except after last word)
                if (i < rawWords.length - 1) {
                    characters.push(" ");
                    wordBoundaries.push(characters.length);
                }
            });

            nextLetter = characters[0];
            charIndex = 0;
        }
    }

    function getCurrentWordStart() {
        // Find the last word boundary before or at the current character
        let boundaryIndex =
            wordBoundaries.findIndex((boundary) => boundary > charIndex) - 1;
        return boundaryIndex >= 0 ? wordBoundaries[boundaryIndex] : 0;
    }

    onMount(() => {
        resetWords();
    });

    function handleCharacter() {
        if (key == "Escape") {
            resetWords();
        } else if (
            key == nextLetter ||
            (key == "Enter" && nextLetter == "Enter\r") ||
            (key == "-" && nextLetter == "—") ||
            (key == "'" && nextLetter == "") ||
            (key == " " && nextLetter == " ")
        ) {
            charIndex++;

            if (charIndex >= characters.length) {
                resetWords();
            } else {
                nextLetter = characters[charIndex];
            }
        } else {
            // On mistake, go back to the start of the current word
            charIndex = getCurrentWordStart();
            nextLetter = characters[charIndex];
        }
    }

    function handleKeydown(event) {
        key = event.key;

        if (key != "Shift" && key != "Alt") {
            if (selection == "prose") {
                handleCharacter();
            } else {
                letterEquals();
            }
            resetBlink();
        }
    }

    function resetBlink() {
        let element = document.getElementById("next-letter");
        element.classList.remove("blink-me");
        void element.offsetWidth;
        element.classList.add("blink-me");
    }
</script>

<div class="typing-wrapper">
    <div class="text-wrapper">
        <div class="text-content">
            {#each characters as char, i}
                {#if i < charIndex}
                    <span class="complete-text">{char}</span>
                {:else if i == charIndex}
                    <span id="next-letter" class="next-letter blink-me"
                        >{nextLetter}</span
                    >
                {:else}
                    <span class="incomplete-text">{char}</span>
                {/if}
            {/each}
        </div>
    </div>
</div>

<svelte:window on:keydown|preventDefault={handleKeydown} />

<style>
    :root {
        --main: #c4c2b9;
    }
    .typing-wrapper {
        margin: auto;
        width: 80%;
        max-width: 800px;
        padding: 20px;
        padding-top: 20px;
        margin-top: 0px;
    }
    .text-wrapper {
        padding: 2rem;
        border-radius: 5px;
        background-color: rgba(0, 0, 0, 0.2);
    }
    .text-content {
        font-family: "Roboto Mono", monospace;
        font-size: 1.5rem;
        line-height: 1.6;
        color: var(--main);
        word-wrap: break-word;
        white-space: pre-wrap;
    }
    .complete-text {
        color: #575757;
    }
    .next-letter {
        color: var(--main);
        display: inline-block;
        min-width: 0.5em;
        line-height: 0.99;
        box-sizing: border-box;
    }
    .author {
        margin-top: 1rem;
        font-style: italic;
        font-size: 1.2rem;
    }
    .blink-me {
        animation: blinker 0.7s step-end 0s infinite;
        background-color: white;
        color: black;
    }

    @keyframes blinker {
        50% {
            background-color: transparent;
            color: white;
        }
    }
</style>
