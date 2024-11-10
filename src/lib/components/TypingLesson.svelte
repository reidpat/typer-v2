<script>
    import { onMount } from "svelte";

    let characters = $state([]);
    let wordBoundaries = $state([]); // Stores the index where each word starts
    let nextLetter = $state("");
    let key = $state("");
    let charIndex = $state(0);
    let selection = $state("prose");
    let {
        lessonContent = $bindable(),
        handleWrongKey = $bindable(),
        newLesson,
        wpm = $bindable(),
        errors = $bindable(),
    } = $props();

    function filterToKeyboardChars(inputString) {
        // Define the regex pattern for allowed characters
        // This includes:
        // - All basic ASCII letters (a-z, A-Z)
        // - Numbers (0-9)
        // - Common punctuation and symbols found on English keyboards
        // - Space and newline characters
        const keyboardCharsPattern =
            /^[a-zA-Z0-9\s`~!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?\n]+$/;

        // Filter the string character by character
        return inputString
            .split("")
            .filter((char) => keyboardCharsPattern.test(char))
            .join("");
    }

    async function resetWords() {
        startTime = Date.now();
        if (selection == "prose") {
            lessonContent = filterToKeyboardChars(lessonContent);
            let rawWords = lessonContent.split(" ");
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

    let startTime = $state()
    let endTime = $state()

    async function computeWPM() {
        if (charIndex == 1) {
            startTime = Date.now() - 1;
        }
		endTime = Date.now();
		let totalTime = (endTime - startTime) / 1000 / 60;
		let totalWords =  charIndex / 5;
		wpm = (totalWords / totalTime).toFixed(2).padStart(5, '0');
		// let _avg_wpm = avg_wpm + Math.round(((wpm - avg_wpm) / total) * 100) / 100;
		// _avg_wpm = Math.round(avg_wpm * 100) / 100;
		// accuracy = Math.round(((totalTyped - attempts) / totalTyped) * 100) / 100;
		// console.log('total ', totalTyped, 'mistakes ', attempts);
		// console.log('accuracy', accuracy);

		// updateWPM(wpm, _avg_wpm, accuracy);
	}

    function handleCharacter() {
        if (key == "Escape") {
            lessonContent = null;
            newLesson();
        } else if (
            key == nextLetter ||
            (key == "Enter" && nextLetter == "Enter\r") ||
            (key == "-" && nextLetter == "—") ||
            (key == "'" && nextLetter == "") ||
            (key == " " && nextLetter == " ")
        ) {
            charIndex++;

            if (charIndex >= characters.length) {
                lessonContent = null;
                newLesson();
            } else {
                nextLetter = characters[charIndex];
            }
        } else {
            // On mistake, go back to the start of the current word
            errors.push(nextLetter);
            charIndex = getCurrentWordStart();
            nextLetter = characters[charIndex];
        }
        computeWPM();
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
            {#each wordBoundaries as boundary, wordIdx}
                {@const nextBoundary =
                    wordBoundaries[wordIdx + 1] || characters.length}
                {@const wordChars = characters.slice(boundary, nextBoundary)}
                {@const isSpace =
                    wordChars.length === 1 && wordChars[0] === " "}
                <span class="word {isSpace ? 'space' : ''}">
                    {#each wordChars as char, i}
                        {@const absoluteIndex = boundary + i}
                        {#if absoluteIndex < charIndex}
                            <span class="complete-text"
                                >{isSpace ? "\u00A0" : char}</span
                            >
                        {:else if absoluteIndex == charIndex}
                            <span id="next-letter" class="next-letter blink-me"
                                >{isSpace ? "\u00A0" : nextLetter}</span
                            >
                        {:else}
                            <span class="incomplete-text"
                                >{isSpace ? "\u00A0" : char}</span
                            >
                        {/if}
                    {/each}
                </span>
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
        max-width: 1200px;
        padding: 20px;
        padding-top: 20px;
        margin-top: 0px;
    }
    .word {
        display: inline-block;
        white-space: nowrap;
    }
    .space {
        white-space: pre;
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
