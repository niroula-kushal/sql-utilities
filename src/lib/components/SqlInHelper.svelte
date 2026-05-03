<script lang="ts">
  type Tool = 'in' | 'converter';
  type Dialect = 'postgresql' | 'mysql' | 'sqlserver';

  let activeTool: Tool = 'in';

  // IN helper state
  let inputText = '';
  let outputText = '';
  let quoteType: 'single' | 'double' = 'single';
  let outputFormat: 'single' | 'multi' = 'multi';
  let prependWithIn = true;

  // converter state
  let sourceDialect: Dialect = 'postgresql';
  let targetDialects: Dialect[] = ['mysql', 'sqlserver'];
  let queryInput = '';
  let convertedOutput = '';

  let copySuccess = false;

  const DIALECTS: { value: Dialect; label: string }[] = [
    { value: 'postgresql', label: 'PostgreSQL' },
    { value: 'mysql', label: 'MySQL' },
    { value: 'sqlserver', label: 'SQL Server' }
  ];

  $: {
    inputText, quoteType, outputFormat, prependWithIn;
    processInInput();
  }

  $: {
    sourceDialect, targetDialects, queryInput;
    processQueryInput();
  }

  function escapeQuotes(text: string) {
    if (quoteType === 'single') {
      return text.replace(/'/g, "\\'");
    }

    return text.replace(/"/g, '\\"');
  }

  function processInInput() {
    if (!inputText.trim()) {
      outputText = '';
      return;
    }

    const lines = inputText
      .split('\n')
      .map((line) => escapeQuotes(line.trim()))
      .filter((line) => line.length > 0);

    if (!lines.length) {
      outputText = '';
      return;
    }

    const quote = quoteType === 'single' ? "'" : '"';
    const formatted = lines.map((line) => `${quote}${line}${quote}`);

    const result = outputFormat === 'single'
      ? `(${formatted.join(', ')})`
      : `(\n  ${formatted.join(',\n  ')}\n)`;

    outputText = prependWithIn ? `IN ${result}` : result;
  }

  function toggleTarget(dialect: Dialect) {
    if (dialect === sourceDialect) return;

    if (targetDialects.includes(dialect)) {
      targetDialects = targetDialects.filter((d) => d !== dialect);
      return;
    }

    targetDialects = [...targetDialects, dialect];
  }

  function normalizeFromSource(query: string, source: Dialect) {
    let normalized = query;

    if (source === 'mysql') {
      normalized = normalized
        .replace(/`([^`]+)`/g, '"$1"')
        .replace(/\bNOW\s*\(\s*\)/gi, 'CURRENT_TIMESTAMP')
        .replace(/\bIFNULL\s*\(/gi, 'COALESCE(')
        .replace(/\bLIMIT\s+(\d+)\s*,\s*(\d+)/gi, 'LIMIT $2 OFFSET $1');
    }

    if (source === 'sqlserver') {
      normalized = normalized
        .replace(/\[([^\]]+)\]/g, '"$1"')
        .replace(/\bGETDATE\s*\(\s*\)/gi, 'CURRENT_TIMESTAMP')
        .replace(/\bISNULL\s*\(/gi, 'COALESCE(');
    }

    return normalized;
  }

  function convertToTarget(query: string, target: Dialect) {
    let converted = query;

    if (target === 'postgresql') return converted;

    if (target === 'mysql') {
      return converted
        .replace(/"([^"]+)"/g, '`$1`')
        .replace(/\bCURRENT_TIMESTAMP\b/gi, 'NOW()')
        .replace(/\bILIKE\b/gi, 'LIKE')
        .replace(/LIMIT\s+(\d+)\s+OFFSET\s+(\d+)/gi, 'LIMIT $2, $1');
    }

    return converted
      .replace(/"([^"]+)"/g, '[$1]')
      .replace(/\bCURRENT_TIMESTAMP\b/gi, 'GETDATE()')
      .replace(/\bTRUE\b/gi, '1')
      .replace(/\bFALSE\b/gi, '0');
  }

  function processQueryInput() {
    const cleanInput = queryInput.trim();

    if (!cleanInput) {
      convertedOutput = '';
      return;
    }

    const normalized = normalizeFromSource(cleanInput, sourceDialect);
    const targets = targetDialects.filter((target) => target !== sourceDialect);

    if (!targets.length) {
      convertedOutput = 'Select at least one target SQL dialect.';
      return;
    }

    convertedOutput = targets
      .map((target) => {
        const title = DIALECTS.find((d) => d.value === target)?.label ?? target;
        return `-- ${title}\n${convertToTarget(normalized, target)}`;
      })
      .join('\n\n');
  }

  async function copyToClipboard(value: string) {
    if (!value) return;

    try {
      await navigator.clipboard.writeText(value);
      copySuccess = true;
      setTimeout(() => (copySuccess = false), 2000);
    } catch (err) {
      console.error('Failed to copy to clipboard:', err);
    }
  }

  function clearAll() {
    if (activeTool === 'in') {
      inputText = '';
      outputText = '';
      return;
    }

    queryInput = '';
    convertedOutput = '';
  }

  function loadSample() {
    if (activeTool === 'in') {
      inputText = `apple\nbanana\ncherry\ndate\nelderberry\nfig\ngrape`;
      return;
    }

    sourceDialect = 'postgresql';
    targetDialects = ['mysql', 'sqlserver'];
    queryInput = `SELECT "id", "name", CURRENT_TIMESTAMP\nFROM "users"\nWHERE "name" ILIKE '%john%'\nORDER BY "id" DESC\nLIMIT 10 OFFSET 5;`;
  }
</script>

<div class="space-y-6">
  <div class="text-center">
    <h2 class="text-3xl font-bold text-gray-900 mb-2">SQL Utilities</h2>
    <p class="text-gray-600 max-w-2xl mx-auto">Use the classic IN helper or convert full queries across SQL dialects.</p>
  </div>

  <div class="card">
    <h3 class="text-lg font-semibold text-gray-900 mb-4">Tool</h3>
    <div class="flex space-x-6">
      <label class="flex items-center"><input type="radio" bind:group={activeTool} value="in" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">SQL IN Helper</span></label>
      <label class="flex items-center"><input type="radio" bind:group={activeTool} value="converter" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Query Converter</span></label>
    </div>
  </div>

  {#if activeTool === 'in'}
    <div class="card">
      <h3 class="text-lg font-semibold text-gray-900 mb-4">IN Output Options</h3>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div><p class="block text-sm font-medium text-gray-700 mb-2">Quote Type</p><div class="flex space-x-4"><label class="flex items-center"><input type="radio" bind:group={quoteType} value="single" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Single</span></label><label class="flex items-center"><input type="radio" bind:group={quoteType} value="double" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Double</span></label></div></div>
        <div><p class="block text-sm font-medium text-gray-700 mb-2">Output Format</p><div class="flex space-x-4"><label class="flex items-center"><input type="radio" bind:group={outputFormat} value="single" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Single line</span></label><label class="flex items-center"><input type="radio" bind:group={outputFormat} value="multi" class="w-4 h-4 text-primary-600 border-gray-300 focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Multi-line</span></label></div></div>
        <div><p class="block text-sm font-medium text-gray-700 mb-2">SQL Clause</p><label class="flex items-center"><input type="checkbox" bind:checked={prependWithIn} class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">Prepend with IN</span></label></div>
      </div>
    </div>
  {:else}
    <div class="card space-y-4">
      <h3 class="text-lg font-semibold text-gray-900">Conversion Settings</h3>
      <div><label for="source-dialect" class="block text-sm font-medium text-gray-700 mb-2">Source Dialect</label><select id="source-dialect" bind:value={sourceDialect} class="textarea-field min-h-0 py-2">{#each DIALECTS as dialect}<option value={dialect.value}>{dialect.label}</option>{/each}</select></div>
      <div><p class="block text-sm font-medium text-gray-700 mb-2">Target Dialects (select one or more)</p><div class="flex flex-wrap gap-4">{#each DIALECTS as dialect}<label class="flex items-center"><input type="checkbox" checked={targetDialects.includes(dialect.value)} disabled={dialect.value === sourceDialect} on:change={() => toggleTarget(dialect.value)} class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500" /><span class="ml-2 text-sm text-gray-700">{dialect.label}</span></label>{/each}</div></div>
    </div>
  {/if}

  <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
    <div class="card">
      <div class="flex justify-between items-center mb-4"><h3 class="text-lg font-semibold text-gray-900">{activeTool === 'in' ? 'Input Data' : 'Input Query'}</h3><div class="flex space-x-2"><button on:click={loadSample} class="btn btn-secondary text-xs">Load Sample</button><button on:click={clearAll} class="btn btn-secondary text-xs">Clear</button></div></div>
      {#if activeTool === 'in'}
        <textarea bind:value={inputText} placeholder="Paste one item per line..." class="textarea-field min-h-[300px] font-mono text-sm"></textarea>
      {:else}
        <textarea bind:value={queryInput} placeholder="Paste a SQL query here..." class="textarea-field min-h-[300px] font-mono text-sm"></textarea>
      {/if}
    </div>
    <div class="card">
      <div class="flex justify-between items-center mb-4"><h3 class="text-lg font-semibold text-gray-900">{activeTool === 'in' ? 'SQL Output' : 'Converted Query'}</h3><button on:click={() => copyToClipboard(activeTool === 'in' ? outputText : convertedOutput)} disabled={!(activeTool === 'in' ? outputText : convertedOutput)} class="btn {copySuccess ? 'btn-success' : 'btn-primary'} text-xs disabled:opacity-50 disabled:cursor-not-allowed">{copySuccess ? 'Copied!' : 'Copy SQL'}</button></div>
      <textarea value={activeTool === 'in' ? outputText : convertedOutput} readonly class="textarea-field min-h-[300px] font-mono text-sm bg-gray-50" placeholder="Generated SQL will appear here..."></textarea>
    </div>
  </div>
</div>
