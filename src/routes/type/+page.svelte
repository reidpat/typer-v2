<script>
    import { onMount } from "svelte";
    import Llm from "../../lib/components/LLM.svelte";
    import TypingLesson from "../../lib/components/TypingLesson.svelte";
    import Stats from "../../lib/components/Stats.svelte";

    let errors = $state([]),
        wpm = $state((0).toFixed(2).padStart(5, "0")),
        accuracy = $state(0),
        focus = $state([]),
        avg_accuracy = $state(0),
        avg_wpm = $state(0);

    function handleWrongKey(key) {
        errors.push(key);
    }

    let generateNew;

    onMount(() => {
        newLesson();
    });

    function readStats() {
        const stats = localStorage.getItem("typingStats")
            ? JSON.parse(localStorage.getItem("typingStats"))
            : { lessons: [] };
        return stats;
    }

    function caluclateAverage(lessons) {
        avg_wpm = 0;
        avg_accuracy = 0;

        for (let i = 0; i < lessons.length; i++) {
            avg_accuracy += parseInt(lessons[i].accuracy);
            avg_wpm += lessons[i].wpm;
        }

        avg_accuracy = (avg_accuracy / lessons.length).toFixed(2);
        avg_wpm = (avg_wpm / lessons.length).toFixed(2);

        
    }

    function newLesson() {
        let stats = saveStats(wpm, accuracy);
        console.log(stats);
        caluclateAverage(stats.lessons);
        console.log(avg_accuracy, avg_wpm);

        content = "";
        focus = errors.toString();
        errors = [];
        generateNew();
    }

    function saveStats(wpm, accuracy) {
        const stats = localStorage.getItem("typingStats")
            ? JSON.parse(localStorage.getItem("typingStats"))
            : { lessons: [] };

        if (accuracy) {
            stats.lessons.push({
                wpm: Number(wpm),
                accuracy: accuracy,
                date: new Date().toISOString(),
            });
        }

        localStorage.setItem("typingStats", JSON.stringify(stats));

        return stats;
    }

    let content = $state([]);
</script>

<div class="m-20">
    <Stats {errors} {wpm} {avg_wpm} {focus} {accuracy} {avg_accuracy}/>
    <div class="divider w-4/5 m-auto"></div>
    <Llm bind:content bind:generateNew bind:focus />
    {#if content.length > 0}
        <TypingLesson
            lessonContent={content}
            {handleWrongKey}
            {newLesson}
            bind:wpm
            bind:errors
            bind:accuracy
        />
        <span class="m-auto w-3/5 text-center flex justify-center items-center gap-4">Press 'esc' to reset with a new lesson</span>
    {/if}
</div>
