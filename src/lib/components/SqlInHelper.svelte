<script lang="ts">
  let inputText = '';
  let outputText = '';
  let quoteType: 'single' | 'double' = 'single';
  let outputFormat: 'single' | 'multi' = 'multi';
  let prependWithIn = true;
  let copySuccess = false;

  function escapeQuotes(text: string) {
    if (quoteType === 'single') {
      return text.replace(/'/g, "\\'");
    } else {
      return text.replace(/"/g, '\\"');
    }
  };

  // Reactive statement to process input whenever input or settings change
  $: {
    // Explicitly reference all dependencies to ensure reactivity
    inputText, quoteType, outputFormat, prependWithIn;
    if (inputText.trim()) {
      processInput();
    } else {
      outputText = '';
    }
  }

  function processInput() {
    if (!inputText.trim()) {
      outputText = '';
      return;
    }

    // Split input into lines and filter out empty lines
    const lines = inputText
      .split('\n')
      .map(line => {
        let final = line.trim();
        return escapeQuotes(final);
      })
      .filter(line => line.length > 0);

    if (lines.length === 0) {
      outputText = '';
      return;
    }
    // Escape quotes in the content based on quote type


    // Apply escaping to all lines
    // Choose quote character based on selection
    const quote = quoteType === 'single' ? "'" : '"';

    // Format each line with quotes
    const formattedLines = lines.map(line => `${quote}${line}${quote}`);

    // Generate output based on format preference
    let result: string;
    if (outputFormat === 'single') {
      result = `(${formattedLines.join(', ')})`;
    } else {
      // Multi-line format
      const indent = '  ';
      result = '(\n' + 
        formattedLines.map(line => `${indent}${line}`).join(',\n') + 
        '\n)';
    }

    // Prepend with IN if option is checked
    outputText = prependWithIn ? `IN ${result}` : result;
  }

  async function copyToClipboard() {
    if (!outputText) return;

    try {
      await navigator.clipboard.writeText(outputText);
      copySuccess = true;
      setTimeout(() => {
        copySuccess = false;
      }, 2000);
    } catch (err) {
      console.error('Failed to copy to clipboard:', err);
    }
  }

  function clearAll() {
    inputText = '';
    outputText = '';
  }

  // Sample data for demo
  function loadSample() {
    inputText = `apple
banana
cherry
date
elderberry
fig
grape`;
  }
</script>

<div class="space-y-6">
  <!-- Header Section -->
  <div class="text-center">
    <h2 class="text-3xl font-bold text-gray-900 mb-2">SQL IN Helper</h2>
    <p class="text-gray-600 max-w-2xl mx-auto">
      Transform your list of values into properly formatted SQL IN clauses.
      Paste your data, choose your formatting options, and get ready-to-use SQL.
    </p>
  </div>

  <!-- Options Panel -->
  <div class="card">
    <h3 class="text-lg font-semibold text-gray-900 mb-4">Output Options</h3>
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <!-- Quote Type Selection -->
      <div>
        <fieldset>
          <legend class="block text-sm font-medium text-gray-700 mb-2">Quote Type</legend>
          <div class="flex space-x-4">
            <label class="flex items-center">
              <input
                type="radio"
                bind:group={quoteType}
                value="single"
                class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500"
              />
              <span class="ml-2 text-sm text-gray-700">Single quotes (')</span>
            </label>
            <label class="flex items-center">
              <input
                type="radio"
                bind:group={quoteType}
                value="double"
                class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500"
              />
              <span class="ml-2 text-sm text-gray-700">Double quotes (")</span>
            </label>
          </div>
        </fieldset>
      </div>

      <!-- Output Format Selection -->
      <div>
        <fieldset>
          <legend class="block text-sm font-medium text-gray-700 mb-2">Output Format</legend>
          <div class="flex space-x-4">
            <label class="flex items-center">
              <input 
                type="radio" 
                bind:group={outputFormat} 
                value="single"
                class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500"
              />
              <span class="ml-2 text-sm text-gray-700">Single line</span>
            </label>
            <label class="flex items-center">
              <input 
                type="radio" 
                bind:group={outputFormat} 
                value="multi"
                class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500"
              />
              <span class="ml-2 text-sm text-gray-700">Multi-line</span>
            </label>
          </div>
        </fieldset>
      </div>

      <!-- Prepend with IN Option -->
      <div>
        <fieldset>
          <legend class="block text-sm font-medium text-gray-700 mb-2">SQL Clause</legend>
          <label class="flex items-center">
            <input 
              type="checkbox" 
              bind:checked={prependWithIn}
              class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500"
            />
            <span class="ml-2 text-sm text-gray-700">Prepend with IN</span>
          </label>
        </fieldset>
      </div>
    </div>
  </div>

  <!-- Input/Output Section -->
  <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
    <!-- Input Panel -->
    <div class="card">
      <div class="flex justify-between items-center mb-4">
        <h3 class="text-lg font-semibold text-gray-900">Input Data</h3>
        <div class="flex space-x-2">
          <button
            on:click={loadSample}
            class="btn btn-secondary text-xs"
          >
            Load Sample
          </button>
          <button
            on:click={clearAll}
            class="btn btn-secondary text-xs"
          >
            Clear
          </button>
        </div>
      </div>

      <textarea
        bind:value={inputText}
        placeholder="Paste your data here, one item per line...&#10;&#10;Example:&#10;apple&#10;banana&#10;cherry"
        class="textarea-field min-h-[300px] font-mono text-sm"
      ></textarea>

      <p class="text-xs text-gray-500 mt-2">
        {inputText.split('\n').filter(line => line.trim()).length} items
      </p>
    </div>

    <!-- Output Panel -->
    <div class="card">
      <div class="flex justify-between items-center mb-4">
        <h3 class="text-lg font-semibold text-gray-900">SQL Output</h3>
        <button
          on:click={copyToClipboard}
          disabled={!outputText}
          class="btn btn-primary text-xs flex items-center space-x-2"
          class:opacity-50={!outputText}
        >
          {#if copySuccess}
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7"></path>
            </svg>
            <span>Copied!</span>
          {:else}
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 16H6a2 2 0 01-2-2V6a2 2 0 012-2h8a2 2 0 012 2v2m-6 12h8a2 2 0 002-2v-8a2 2 0 00-2-2h-8a2 2 0 00-2 2v8a2 2 0 002 2z"></path>
            </svg>
            <span>Copy</span>
          {/if}
        </button>
      </div>

      <div class="relative">
        <pre class="code-block min-h-[300px] whitespace-pre-wrap">{outputText || 'Your formatted SQL will appear here...'}</pre>
      </div>
    </div>
  </div>

  <!-- Usage Example -->
  <div class="card bg-blue-50 border-blue-200">
    <h3 class="text-lg font-semibold text-blue-900 mb-3">Usage Example</h3>
    <div class="space-y-3">
      <div>
        <p class="text-sm text-blue-800 mb-2"><strong>Input:</strong></p>
        <pre class="bg-white p-2 rounded border text-xs font-mono">apple
banana
cherry</pre>
      </div>
      <div>
        <p class="text-sm text-blue-800 mb-2"><strong>Output (Multi-line):</strong></p>
        <pre class="bg-white p-2 rounded border text-xs font-mono">IN (
  'apple',
  'banana',
  'cherry'
)</pre>
      </div>
      <div>
        <p class="text-sm text-blue-800 mb-2"><strong>Use in SQL:</strong></p>
        <pre class="bg-white p-2 rounded border text-xs font-mono">SELECT * FROM fruits WHERE name IN (
  'apple',
  'banana',
  'cherry'
);</pre>
      </div>
    </div>
  </div>
</div>
