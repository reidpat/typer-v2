<script>
    import { onMount } from "svelte";
    import { PUBLIC_OPEN_AI_API_KEY } from "$env/static/public";

    let { content = $bindable(), generateNew = $bindable(), focus } = $props();

    let generalInstructions =
        " It should only contain characters found on the english keyboard";

    async function generate() {
        console.log(focus);
        let prompt;
        let maxTokens = 100
        if (focus.length > 1) {
            maxTokens = 30;
            prompt = `Generate a typing test for the user which focus on building skill with the ${focus} keys. The test should be in the form of full words, but need not be full sentences, or make sense. It should not include commas between each word. Your response should only contain the test, with no other words or characters`;
        } else {
            prompt = `Generate a four to five sentence response designed as a typing test for the user. This test should be unique, creative, inspirational, and educational. Include no other words or characters.`;
        }
        console.log(prompt);
        let res = await fetch("https://api.openai.com/v1/chat/completions", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
                Authorization: `Bearer ${PUBLIC_OPEN_AI_API_KEY}`,
            },
            body: JSON.stringify({
                model: "gpt-4o-mini",
                messages: [
                    {
                        role: "user",
                        content: prompt + generalInstructions,
                    },
                ],
                temperature: 0.8,
                max_completion_tokens: maxTokens,
            }),
        });
        let data = await res.json();
        content = data.choices[0].message.content;
        console.log(data);
    }

    onMount(async () => {
        generate();

        generateNew = async () => await generate();
    });
</script>
