<script lang="ts">
  type Dialect = 'postgresql' | 'mysql' | 'sqlserver';

  let sourceDialect: Dialect = 'postgresql';
  let targetDialects: Dialect[] = ['mysql', 'sqlserver'];
  let inputText = '';
  let outputText = '';
  let copySuccess = false;

  const DIALECTS: { value: Dialect; label: string }[] = [
    { value: 'postgresql', label: 'PostgreSQL' },
    { value: 'mysql', label: 'MySQL' },
    { value: 'sqlserver', label: 'SQL Server' }
  ];

  $: {
    sourceDialect, targetDialects, inputText;
    processInput();
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

    if (target === 'postgresql') {
      return converted;
    }

    if (target === 'mysql') {
      converted = converted
        .replace(/"([^"]+)"/g, '`$1`')
        .replace(/\bCURRENT_TIMESTAMP\b/gi, 'NOW()')
        .replace(/\bILIKE\b/gi, 'LIKE')
        .replace(/LIMIT\s+(\d+)\s+OFFSET\s+(\d+)/gi, 'LIMIT $2, $1');
      return converted;
    }

    converted = converted
      .replace(/"([^"]+)"/g, '[$1]')
      .replace(/\bCURRENT_TIMESTAMP\b/gi, 'GETDATE()')
      .replace(/\bTRUE\b/gi, '1')
      .replace(/\bFALSE\b/gi, '0');

    return converted;
  }

  function processInput() {
    const cleanInput = inputText.trim();

    if (!cleanInput) {
      outputText = '';
      return;
    }

    const normalized = normalizeFromSource(cleanInput, sourceDialect);
    const effectiveTargets = targetDialects.filter((target) => target !== sourceDialect);

    if (effectiveTargets.length === 0) {
      outputText = 'Select at least one target SQL dialect.';
      return;
    }

    outputText = effectiveTargets
      .map((target) => {
        const title = DIALECTS.find((d) => d.value === target)?.label ?? target;
        return `-- ${title}\n${convertToTarget(normalized, target)}`;
      })
      .join('\n\n');
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
    sourceDialect = 'postgresql';
    targetDialects = ['mysql', 'sqlserver'];
    inputText = `SELECT "id", "name", CURRENT_TIMESTAMP
FROM "users"
WHERE "name" ILIKE '%john%'
ORDER BY "id" DESC
LIMIT 10 OFFSET 5;`;
  }
</script>

<div class="space-y-6">
  <div class="text-center">
    <h2 class="text-3xl font-bold text-gray-900 mb-2">SQL Query Converter</h2>
    <p class="text-gray-600 max-w-2xl mx-auto">Convert SQL queries between PostgreSQL, MySQL, and SQL Server.</p>
  </div>

  <div class="card space-y-4">
    <h3 class="text-lg font-semibold text-gray-900">Conversion Settings</h3>

    <div>
      <label for="source-dialect" class="block text-sm font-medium text-gray-700 mb-2">Source Dialect</label>
      <select id="source-dialect" bind:value={sourceDialect} class="textarea-field min-h-0 py-2">
        {#each DIALECTS as dialect}
          <option value={dialect.value}>{dialect.label}</option>
        {/each}
      </select>
    </div>

    <div>
      <p class="block text-sm font-medium text-gray-700 mb-2">Target Dialects (select one or more)</p>
      <div class="flex flex-wrap gap-4">
        {#each DIALECTS as dialect}
          <label class="flex items-center">
            <input
              type="checkbox"
              checked={targetDialects.includes(dialect.value)}
              disabled={dialect.value === sourceDialect}
              on:change={() => toggleTarget(dialect.value)}
              class="w-4 h-4 text-primary-600 border-gray-300 rounded focus:ring-primary-500"
            />
            <span class="ml-2 text-sm text-gray-700">{dialect.label}</span>
          </label>
        {/each}
      </div>
    </div>
  </div>

  <div class="grid grid-cols-1 lg:grid-cols-2 gap-6">
    <div class="card">
      <div class="flex justify-between items-center mb-4">
        <h3 class="text-lg font-semibold text-gray-900">Input Query</h3>
        <div class="flex space-x-2">
          <button on:click={loadSample} class="btn btn-secondary text-xs">Load Sample</button>
          <button on:click={clearAll} class="btn btn-secondary text-xs">Clear</button>
        </div>
      </div>
      <textarea bind:value={inputText} placeholder="Paste a SQL query here..." class="textarea-field min-h-[300px] font-mono text-sm"></textarea>
    </div>

    <div class="card">
      <div class="flex justify-between items-center mb-4">
        <h3 class="text-lg font-semibold text-gray-900">Converted Query</h3>
        <button on:click={copyToClipboard} disabled={!outputText} class="btn {copySuccess ? 'btn-success' : 'btn-primary'} text-xs disabled:opacity-50 disabled:cursor-not-allowed">{copySuccess ? 'Copied!' : 'Copy SQL'}</button>
      </div>
      <textarea value={outputText} readonly class="textarea-field min-h-[300px] font-mono text-sm bg-gray-50" placeholder="Converted SQL will appear here..."></textarea>
    </div>
  </div>
</div>
