<script>
    let { wpm = 0, errors = [], focus = ""} = $props();
</script>

<div class="m-auto w-3/5 flex justify-start items-center gap-4">
    <div>WPM: {wpm}</div>
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
