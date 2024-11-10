<script>
    import { onMount } from "svelte";
    import Llm from "../../lib/components/LLM.svelte";
    import TypingLesson from "../../lib/components/TypingLesson.svelte";
    import Stats from "../../lib/components/Stats.svelte";

    let errors = $state([]),
        wpm = $state((0).toFixed(2).padStart(5, "0")),
        focus = $state([]);

    function handleWrongKey(key) {
        errors.push(key);
    }

    let generateNew;

    onMount(() => {
        newLesson();
    });

    function newLesson() {
        content = "";
        focus = errors.toString();
        errors = [];
        generateNew();
    }

    let content = $state([]);
</script>

<div class="m-20">
    <Stats {errors} {wpm} {focus} />
    <div class="divider w-4/5 m-auto"></div>
    <Llm bind:content bind:generateNew {focus} />
    {#if content.length > 0}
        <TypingLesson
            lessonContent={content}
            {handleWrongKey}
            {newLesson}
            bind:wpm
            bind:errors
        />
    {/if}
</div>
