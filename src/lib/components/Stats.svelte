<script>
    let { wpm = 0, avg_wpm, errors = [], focus = "", accuracy = 0, avg_accuracy} = $props();


    let wpmDiff = $derived((wpm - avg_wpm).toFixed(2))
    let accuracyDiff = $derived((accuracy - avg_accuracy).toFixed(2))

    let wpmColor = $derived(wpmDiff > 0 ? "text-green-300" : "text-red-300")
    let accuracyColor = $derived(accuracyDiff > 0 ? "text-green-300" : "text-red-300")

    let wpmDiffText = $derived(wpmDiff > 0 ? `+${wpmDiff}` : wpmDiff)
    let accuracyDiffText = $derived(accuracyDiff > 0 ? `+${accuracyDiff}` : accuracyDiff)

</script>

<div class="m-auto w-3/5 flex justify-start items-center gap-4">
    <div>WPM: {wpm} <span class="{wpmColor}">({wpmDiffText})</span></div>
    <div>Accuracy: {accuracy} <span class="{accuracyColor}">({accuracyDiffText})</span></div>

    <div>
        <span>Errors: </span>      
            <span style="color: red;">{errors.toString()}</span>
    </div>
    <div>
        <span
            >Focus:
            <span class="text-green-300">{focus}</span>
        </span>
    </div>
</div>
