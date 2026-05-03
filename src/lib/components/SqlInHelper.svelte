<script lang="ts">
  let toolType: 'in' | 'between' = 'in';
  let inputText = '';
  let outputText = '';
  let quoteType: 'single' | 'double' = 'single';
  let outputFormat: 'single' | 'multi' = 'multi';
  let prependWithIn = true;
  let prependWithBetween = true;
  let useNotBetween = false;
  let copySuccess = false;

  function escapeQuotes(text: string) {
    if (quoteType === 'single') {
      return text.replace(/'/g, "\\'");
    }
    return text.replace(/"/g, '\\"');
  }

  $: {
    toolType, inputText, quoteType, outputFormat, prependWithIn, prependWithBetween, useNotBetween;
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

    const quote = quoteType === 'single' ? "'" : '"';

    if (toolType === 'in') {
      const lines = inputText
        .split('\n')
        .map((line) => escapeQuotes(line.trim()))
        .filter((line) => line.length > 0);

      if (lines.length === 0) {
        outputText = '';
        return;
      }

      const formattedLines = lines.map((line) => `${quote}${line}${quote}`);

      let result: string;
      if (outputFormat === 'single') {
        result = `(${formattedLines.join(', ')})`;
      } else {
        const indent = '  ';
        result = '(\n' + formattedLines.map((line) => `${indent}${line}`).join(',\n') + '\n)';
      }

      outputText = prependWithIn ? `IN ${result}` : result;
      return;
    }

    const values = inputText
      .split(/\n|,/) 
      .map((v) => escapeQuotes(v.trim()))
      .filter((v) => v.length > 0);

    if (values.length < 2) {
      outputText = 'Please provide at least two values (start and end).';
      return;
    }

    const start = `${quote}${values[0]}${quote}`;
    const end = `${quote}${values[1]}${quote}`;
    const betweenClause = `${useNotBetween ? 'NOT BETWEEN' : 'BETWEEN'} ${start} AND ${end}`;
    outputText = prependWithBetween ? betweenClause : `${start} AND ${end}`;
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

  function loadSample() {
    if (toolType === 'in') {
      inputText = `apple\nbanana\ncherry\ndate\nelderberry\nfig\ngrape`;
      return;
    }

    inputText = `2024-01-01\n2024-12-31`;
  }
</script>

<div class="space-y-6">
  <div class="text-center">
    <h2 class="text-3xl font-bold text-gray-900 mb-2">SQL Query Converter</h2>
    <p class="text-gray-600 max-w-2xl mx-auto">
      Convert values into SQL <code>IN</code> or <code>BETWEEN</code> query snippets.
    </p>
  </div>

  <div class="card">
    <h3 class="text-lg font-semibold text-gray-900 mb-4">Tool</h3>
    <div class="flex space-x-6">
      <label class="flex items-center">
        <input type="radio" bind:group={toolType} value="in" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" />
        <span class="ml-2 text-sm text-gray-700">IN Helper</span>
      </label>
      <label class="flex items-center">
        <input type="radio" bind:group={toolType} value="between" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" />
        <span class="ml-2 text-sm text-gray-700">BETWEEN Helper</span>
      </label>
    </div>
  </div>

  <div class="card">
    <h3 class="text-lg font-semibold text-gray-900 mb-4">Output Options</h3>
    <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
      <div>
        <fieldset>
          <legend class="block text-sm font-medium text-gray-700 mb-2">Quote Type</legend>
          <div class="flex space-x-4">
            <label class="flex items-center"><input type="radio" bind:group={quoteType} value="single" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Single</span></label>
            <label class="flex items-center"><input type="radio" bind:group={quoteType} value="double" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Double</span></label>
          </div>
        </fieldset>
      </div>

      {#if toolType === 'in'}
        <div>
          <fieldset>
            <legend class="block text-sm font-medium text-gray-700 mb-2">Output Format</legend>
            <div class="flex space-x-4">
              <label class="flex items-center"><input type="radio" bind:group={outputFormat} value="single" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Single line</span></label>
              <label class="flex items-center"><input type="radio" bind:group={outputFormat} value="multi" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Multi-line</span></label>
            </div>
          </fieldset>
        </div>
        <div><fieldset><legend class="block text-sm font-medium text-gray-700 mb-2">SQL Clause</legend><label class="flex items-center"><input type="checkbox" bind:checked={prependWithIn} class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Prepend with IN</span></label></fieldset></div>
      {:else}
        <div><fieldset><legend class="block text-sm font-medium text-gray-700 mb-2">BETWEEN Type</legend><label class="flex items-center"><input type="checkbox" bind:checked={useNotBetween} class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Use NOT BETWEEN</span></label></fieldset></div>
        <div><fieldset><legend class="block text-sm font-medium text-gray-700 mb-2">SQL Clause</legend><label class="flex items-center"><input type="checkbox" bind:checked={prependWithBetween} class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Include BETWEEN keyword</span></label></fieldset></div>
      {/if}
    </div>
  </div>

  <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
    <div class="card">
      <div class="flex justify-between items-center mb-4"><h3 class="text-lg font-semibold text-gray-900">Input Data</h3><div class="flex space-x-2"><button on:click={loadSample} class="btn btn-secondary text-xs">Load Sample</button><button on:click={clearAll} class="btn btn-secondary text-xs">Clear</button></div></div>
      <textarea bind:value={inputText} placeholder={toolType === 'in' ? "Paste one item per line..." : "Provide start and end (one per line or comma-separated)..."} class="textarea-field min-h-[300px] font-mono text-sm"></textarea>
    </div>
    <div class="card">
      <div class="flex justify-between items-center mb-4"><h3 class="text-lg font-semibold text-gray-900">SQL Output</h3><button on:click={copyToClipboard} disabled={!outputText} class="btn {copySuccess ? 'btn-success' : 'btn-primary'} text-xs disabled:opacity-50 disabled:cursor-not-allowed">{copySuccess ? 'Copied!' : 'Copy SQL'}</button></div>
      <textarea value={outputText} readonly class="textarea-field min-h-[300px] font-mono text-sm bg-gray-50" placeholder="Generated SQL will appear here..."></textarea>
    </div>
  </div>
</div>
