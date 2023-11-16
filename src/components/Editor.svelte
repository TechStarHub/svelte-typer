<script>
    import Icon from '@iconify/svelte'
    import TyperResult from './TyperResult.svelte';

    const givenText = "SvelteKit provides a filesystem router, server-side rendering (SSR), code-splitting, and hot module replacement (HMR) out of the box. It's built on top of Svelte and Node.js, and is compatible with the vast ecosystem of Node libraries.";

    let typedText = "";
    let wpm = 0;
    let wpms = [];
    let maxWpm = 0;
    let accuracy = "00.00";
    let time = 120;
    let timeElapsed = 0;
    let inputDisabled = true;
    let isCompleted = false;
    let isPaused = false;
    let isStarted = false;
    
    // create an array of letter objects from the givenText
    /* @params text : string */
    function createLetters (text) {
        const letters = text.split("");
        const result = [];
        letters.forEach((letter,index) => {
            result.push({letter:letter,isCorrect: undefined,idx: index});
        });
        return result;
    }
    // handle the input event
    function handleOnchange (event) {
        const { value } = event.target;
        typedText = value;
        checkLetter(typedText);
        // console.log(typedText);
    }
    // check the correctness of the letters 
    function checkLetter (typedText) {
        const letters = typedText.split("");
        let idx = 0;

        // for checking the correctness of the letters
        letters.forEach((letter,index) => {
            idx = index;
            if (letter == givenLetters[index].letter) {
                // console.log(true,letter,givenLetters[index].letter);
                givenLetters[index].isCorrect = true;
            } else {
                givenLetters[index].isCorrect = false;
                // console.log(false,letter,givenLetters[index].letter);
            }
        });

        // for reseting the isCorrect property of the givenLetters
        for (let i = idx+1; i < givenLetters.length; i++) {
            givenLetters[i].isCorrect = undefined;
        }
        // handle the case when the user deletes the text
        if(letters.length === 0){
            givenLetters[0].isCorrect = undefined;
        }

        // check for completion of the text
        if (givenLetters.length === letters.length || time === 0) {
            isCompleted = true;
            inputDisabled = true;
            console.log("completed");
        }

        // calculate accuracy
        let correctLetters = 0;
        givenLetters.forEach(letter => {
            if (letter.isCorrect === true) {
                correctLetters++;
            }
        });
        const calc = ((correctLetters/typedText.length)*100).toFixed(2);
        accuracy = calc === "NaN" ? "00.00" : calc;
        // calculate wpm
    }

    // handle on click event of the resume button
    function handleBtnClick () {
        if (!isStarted && !isPaused) {
            isStarted = true;
            inputDisabled = false;
            return;
        }
        isPaused = !isPaused;
        inputDisabled = !inputDisabled;
    }

    // time interval for the timer
    const interval = setInterval(
        () => {
            if (time === 0 || isCompleted) {
                clearInterval(interval);
                inputDisabled = true;
                isCompleted = true;
                return;
            }
            if (!isPaused && isStarted) {
                time--;
                timeElapsed++;
                wpm = calculateWpm();
                wpms.push(wpm);
                if (wpm > maxWpm) {
                    maxWpm = wpm;
                }
            }
            
        },1000
    )
    // time to string
    function timeToString (time) {
        const minutes = Math.floor(time/60);
        const seconds = time%60;
        return `${ minutes/10 < 1 ? `0${minutes}` : minutes}:${seconds/10<1 ? `0${seconds}` : seconds}`;
    }

    // wpm calculating function
    function calculateWpm () {
        const words = typedText.split(" ").length;
        const minutes = timeElapsed/60;
        const wpm = Math.floor(words/minutes);
        return wpm;
    }


    const givenLetters = createLetters(givenText);
</script>

<div class="p-4 bg-[#EEF5FF] rounded-md shadow-md w-[90vw] sm:w-[80vw] md:w-[60vw]">
    <div class="mb-2">
        <h3 class="text-2xl font-bold text-center mb-2">Given Text</h3>
        <p class="text-xl">
            {#each givenLetters as letter}
                {#if letter.isCorrect === true}
                    <span class="letter text-green-500 ">{letter.letter}</span>
                {:else if letter.isCorrect === false}
                    <span class="letter text-red-500">{letter.letter}</span>
                {:else}
                    <span class="lette">{letter.letter}</span>
                {/if}
            {/each}
        </p>
    </div>
    <div class="">
        <div class="flex justify-between items-center">
            <h3 class="text-2xl font-bold mb-3">Write Here</h3>
            <h3 class="text-2xl font-medium mb-3">Time: {timeToString(time)}</h3>
        </div>
        <textarea disabled={inputDisabled} on:input={handleOnchange} class="w-full rounded border-[#83A2FF]  outline outline-[#83A2FF] p-1" name="" id="" cols="40" rows="6" ></textarea>

        <div class="w-full flex justify-between items-center">
            <h4 class="font-medium">Accuracy : {accuracy} %</h4>
            <h4 class="font-medium">WPM : {wpm} </h4>
        </div>

        {#if isCompleted}
            <h1 class="text-3xl text-center">Completed!!</h1>
            <TyperResult data={{wpm,wpms,accuracy,maxWpm,timeElapsed,time}} />
        {:else}
            <div class="flex items-center justify-center  ">
                <button on:click={handleBtnClick} class="bg-[#83A2FF] text-white px-4 py-1 text-xl rounded-md shadow-md flex items-center justify-center gap-1" >
                    {#if !isStarted}
                        <Icon icon="ph:play-fill" class="w-6 h-6" />
                        <span>Start</span>
                    {:else if isPaused}
                        <Icon icon="ph:play-fill" class="w-6 h-6" />
                        <span>Resume</span>
                    {:else}
                        <Icon icon="ic:round-pause" class="w-7 h-7" />
                        <span>Pause</span>                        
                    {/if}
                </button>
            </div>
        {/if}
    </div>
</div>