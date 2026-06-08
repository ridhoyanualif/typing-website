<script lang="ts">
    import { wordsPool } from "../lib/words";

    // --- State Variables ---
    let currentStep = $state("start"); // "start", "countdown", "test", "result"
    let timeLeft = $state(5.0);
    
    // Test Duration Configuration
    let selectedDuration = $state(30); // Default to 30s
    let testTime = $state(30.0);       // The active ticking countdown
    
    // Typing Stats
    let currentWord = $state("");
    let userInput = $state("");
    let correctWordCount = $state(0);
    let totalErrors = $state(0);

    // --- Helper Timer Formatter ---
    function formatTime(valueInSeconds: number): string {
        const seconds = Math.max(0, valueInSeconds) % 60;
        // If there are minutes remaining, you can extract them here
        const minutes = Math.floor(Math.max(0, valueInSeconds) / 60);
        
        if (minutes > 0) {
            return `${minutes}:${seconds < 10 ? "0" : ""}${seconds.toFixed(2)}`;
        }
        return seconds.toFixed(2);
    }

    // --- Game Logic Functions ---
    function startCountdown() {
        currentStep = "countdown";
        timeLeft = 5.0;
        
        const interval = setInterval(() => {
            timeLeft -= 0.01;
            if (timeLeft <= 0) {
                clearInterval(interval);
                startTest();
            }
        }, 10);
    }

    function generateNewWord() {
        const remainingWords = wordsPool.filter(w => w !== currentWord);
        currentWord = remainingWords[Math.floor(Math.random() * remainingWords.length)];
        userInput = "";
    }

    function startTest() {
        currentStep = "test";
        // Convert string value from select element to number
        testTime = Number(selectedDuration);
        correctWordCount = 0;
        totalErrors = 0;
        generateNewWord();

        const interval = setInterval(() => {
            testTime -= 0.01;
            if (testTime <= 0) {
                clearInterval(interval);
                endTest();
            }
        }, 10);
    }

    function handleKeyDown(event: KeyboardEvent) {
        if (currentStep !== "test") return;

        // 1. SPACEBAR: Move to next word no matter what
        if (event.key === " ") {
            event.preventDefault(); // Stop page scrolling
            
            if (userInput === currentWord) {
                correctWordCount++;
            }
            
            generateNewWord();
            return;
        }

        // 2. Handle Backspace
        if (event.key === "Backspace") {
            userInput = userInput.slice(0, -1);
            return;
        }

        // 3. Ignore non-character keys (Shift, CapsLock, etc.)
        if (event.key.length !== 1) return;

        // 4. Block typing beyond the word's maximum length
        if (userInput.length >= currentWord.length) {
            return;
        }

        // 5. Error Tracking: Check if the newly typed letter is wrong
        const nextCharIndex = userInput.length;
        if (event.key !== currentWord[nextCharIndex]) {
            totalErrors++;
        }

        // Append character
        userInput += event.key;
    }

    function endTest() {
        currentStep = "result";
    }

    function rerollTest() {
        currentStep = "start";
    }

    // Dynamic WPM Calculation: (Correct Words / (Time in Seconds / 60))
    let wpm = $derived(
        correctWordCount > 0 
            ? Math.round(correctWordCount / (Number(selectedDuration) / 60)) 
            : 0
    );
</script>

<svelte:window onkeydown={handleKeyDown} />

{#if currentStep === "start"}
    <div class="w-full h-screen flex justify-center items-center gap-2 text-xl">
        <div class="flex items-center gap-2 bg-gray-100 p-1 rounded-md border border-gray-200">
            <select bind:value={selectedDuration} class="rounded-md py-1 px-2 bg-white border border-gray-300 outline-none cursor-pointer font-semibold">
                <option value={15}>15s</option>
                <option value={30}>30s</option>
                <option value={45}>45s</option>
                <option value={60}>60s</option>
                <option value={120}>120s</option>
                <option value={180}>180s</option>
                <option value={240}>240s</option>
                <option value={300}>300s</option>
            </select>
        </div>

        <button
            onclick={startCountdown}
            class="p-2 px-3 rounded-md bg-black text-white hover:bg-black/80 transition font-semibold"
            type="button">start</button
        >
    </div>
{/if}

{#if currentStep === "countdown"}
    <div class="w-full h-screen flex flex-col gap-4 items-center justify-center">
        <span class="text-4xl">begin in..</span>
        <p class="text-7xl font-mono">{formatTime(timeLeft)}</p>
    </div>
{/if}

{#if currentStep === "test"}
    <div class="w-full h-screen flex items-center justify-center relative">
        <p class="text-4xl absolute top-10 left-10 font-mono">{correctWordCount}</p>
        <p class="text-4xl absolute top-10 right-10 font-mono">{formatTime(testTime)}</p>
        
        <div class="text-7xl font-mono tracking-wide">
            {#each currentWord.split("") as letter, index}
                {#if index < userInput.length}
                    {#if userInput[index] === letter}
                        <span class="text-black">{letter}</span>
                    {:else}
                        <span class="text-red-400">{letter}</span>
                    {/if}
                {:else}
                    <span class="text-gray-400">{letter}</span>
                {/if}
            {/each}
        </div>
    </div>
{/if}

{#if currentStep === "result"}
    <div class="w-full h-screen flex flex-col items-center justify-center gap-10">
        <p class="text-7xl font-bold">({wpm} WPM)</p>
        <p class="text-4xl text-gray-600">
            <span class="text-black font-semibold">{correctWordCount}</span> words and 
            <span class="text-red-500 font-semibold">{totalErrors}</span> errors in {selectedDuration} seconds.
        </p>
        <button onclick={rerollTest}
            class="p-1.5 px-3 text-xl rounded-md bg-black text-white hover:bg-black/80 transition font-semibold"
            type="button">reroll</button
        >
    </div>
{/if}

<span class="absolute bottom-5 right-5 text-black text-2xl text-gray-400">this is a typing test, hit your spacebar to jump to the next word | <a href="" class="hover:text-black transition font-mono underline">r-type</a></span>