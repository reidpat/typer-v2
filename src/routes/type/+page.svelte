<script>
    import { onMount } from "svelte";
    import Llm from "../../lib/components/LLM.svelte";
    import TypingLesson from "../../lib/components/TypingLesson.svelte";
    // let content =
    //     "Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit, sed quia non numquam eius modi tempora incidunt ut labore et dolore magnam aliquam quaerat voluptatem. Ut enim ad minima veniam, quis nostrum exercitationem ullam corporis suscipit laboriosam, nisi ut aliquid ex ea commodi consequatur? Quis autem vel eum iure reprehenderit qui in ea voluptate velit esse quam nihil molestiae consequatur, vel illum qui dolorem eum fugiat quo voluptas nulla pariatur?";

    let errors = $state([]);
    function handleWrongKey(key) {
        errors.push(key);
    }

    let generateNew;

    let focus = $state("");

    onMount(()=>{
        newLesson();
    })

    function newLesson(){
        content = "";  
        focus = errors.toString();
        errors = [];     
        generateNew();
        
    }

    let content = $state([]);
</script>

<div class="flex justify-evenly items-center">
    <div>
    <span>Errors: </span>
    {#each errors as error, i}
    {#if i > 0}
            <span>, </span>
        {/if}
        <span style="color: red;">{error}</span>
        
    {/each}
    </div>
    <div>
        <span>Focus: 
           <span class="text-green-300">{focus}</span> 
        </span>
    </div>
</div>
<Llm bind:content={content} bind:generateNew {focus}/>
{#if content.length > 0}
<TypingLesson lessonContent={content} {handleWrongKey} {newLesson}/>
{/if}
