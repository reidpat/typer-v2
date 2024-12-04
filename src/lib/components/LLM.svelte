<script>
    import { onMount } from "svelte";
    import { PUBLIC_OPEN_AI_API_KEY } from "$env/static/public";

    let { content = $bindable(), generateNew = $bindable(), focus = $bindable()} = $props();

    let generalInstructions =
        " It should only contain characters found on the english keyboard.";

    import keyboards from "$lib/keyboards.json";

    let keyboardData = keyboards[0]; // Assuming we want the first keyboard configuration
    let fingers = keyboardData.fingers;
    // Get the keys contained in the list of the random finger
   
    let loading = $state(false);
    async function generate() {
        const randomFinger = () => {
            const fingerKeys = Object.keys(fingers);
            const randomIndex = Math.floor(Math.random() * fingerKeys.length);
            return fingerKeys[randomIndex];
        };

        let selectedFinger = randomFinger();
        let selectedFingerKeys = fingers[selectedFinger];

        console.log(selectedFingerKeys);

        function generateRandomString() {
            const letters = 'abcdefghijklmnopqrstuvwxyz';
            let result = '';
            for (let i = 0; i < 2; i++) {
                const randomIndex = Math.floor(Math.random() * letters.length);
                result += letters[randomIndex];
            }
            return result;
        }


        loading = true;
        let prompt;
        let maxTokens = 100;
        if (Math.random() > 1) {
            maxTokens = 50;
            focus = selectedFingerKeys;
            prompt = `Generate a typing test for the user which focus on building skill with the ${focus.join("")} keys. The test should take the form of short 'words' of 5-8 characters (only using the focused characters, but using ALL of them) with a space between each short string which get the user to practice moving between these finger keys. Your response should only contain the test, with no other words or characters`;
        }else if (focus.length > 1) {
            maxTokens = 30;
            prompt = `Generate a typing test for the user which focus on building skill with the ${focus} keys. The test should be in the form of full words, but need not be full sentences, or make sense. It should not include commas between each word. Your response should only contain the test, with no other words or characters`;
        } else {
            prompt = `Generate a four to five sentence response designed as a typing test for the user. This test should be unique, inspirational, and educational. It should start with a word that contains the letters '${generateRandomString()}'". Include no other words or characters.`;
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
                temperature: 0.5,
                max_completion_tokens: maxTokens,
            }),
        });
        let data = await res.json();
        content = data.choices[0].message.content.replace(/\s{2,}/g, ' ');
        loading = false;
        console.log(data);
    }

    onMount(async () => {
        generate();

        generateNew = async () => await generate();
    });
</script>

{#if loading}
    <div
        class="w-4/5 m-auto flex flex-col justify-center items-center gap-2 p-4"
    >
        <div class="skeleton h-6 w-full"></div>
        <div class="skeleton h-6 w-full"></div>
    </div>
{/if}
