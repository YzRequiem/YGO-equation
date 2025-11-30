<script>
  let totalCards = $state(10);
  let targetLevel = $state(6);
  let target1 = $state(4);
  let target2 = $state(8);
  let useSecondTarget = $state(true);

  // Panel 1: Cartes → Cibles
  let validCombinations = $derived(() => {
    const results = [];
    
    for (let fusionLevel = 1; fusionLevel <= 12; fusionLevel++) {
      const remainder = totalCards - fusionLevel;
      if (remainder > 0 && remainder % 2 === 0) {
        const xyzRank = remainder / 2;
        if (xyzRank >= 1 && xyzRank <= 12) {
          const targetReachable = fusionLevel + xyzRank;
          
          if (targetReachable <= 12) {
            results.push({
              fusionLevel,
              xyzRank,
              target: targetReachable,
            });
          }
        }
      }
    }
    
    return results.sort((a, b) => a.target - b.target);
  });

  // Grouper par cible pour Panel 1
  let groupedByTarget = $derived(() => {
    const groups = {};
    validCombinations().forEach(combo => {
      if (!groups[combo.target]) {
        groups[combo.target] = [];
      }
      groups[combo.target].push(combo);
    });
    return groups;
  });

  let targets = $derived(() => {
    return Object.keys(groupedByTarget()).map(Number).sort((a, b) => a - b);
  });

  // Panel 2: Cible → Prérequis
  let prerequisByTarget = $derived(() => {
    const results = [];
    
    for (let fusionLevel = 1; fusionLevel < targetLevel && fusionLevel <= 12; fusionLevel++) {
      const xyzRank = targetLevel - fusionLevel;
      if (xyzRank >= 1 && xyzRank <= 12) {
        const totalNeeded = fusionLevel + (2 * xyzRank);
        results.push({
          fusionLevel,
          xyzRank,
          totalNeeded,
        });
      }
    }
    
    return results.sort((a, b) => a.totalNeeded - b.totalNeeded);
  });

  // Panel 3: Extra Deck Builder
  let extraDeckBuilder = $derived(() => {
    const targetsList = useSecondTarget ? [target1, target2] : [target1];
    const allCombos = [];
    
    targetsList.forEach(target => {
      for (let fusionLevel = 1; fusionLevel < target && fusionLevel <= 12; fusionLevel++) {
        const xyzRank = target - fusionLevel;
        if (xyzRank >= 1 && xyzRank <= 12) {
          const totalNeeded = fusionLevel + (2 * xyzRank);
          allCombos.push({
            target,
            fusionLevel,
            xyzRank,
            totalNeeded,
          });
        }
      }
    });

    const fusionLevels = new Set();
    const xyzRanks = new Map();
    
    allCombos.forEach(combo => {
      fusionLevels.add(combo.fusionLevel);
      xyzRanks.set(combo.xyzRank, Math.max(xyzRanks.get(combo.xyzRank) || 0, 2));
    });

    const extraDeck = [];
    
    [...fusionLevels].sort((a, b) => a - b).forEach(level => {
      extraDeck.push({ type: 'Fusion', level, rank: null, label: `Fusion Niv.${level}` });
    });
    
    [...xyzRanks.keys()].sort((a, b) => a - b).forEach(rank => {
      extraDeck.push({ type: 'Xyz', level: null, rank, label: `Xyz Rang ${rank}` });
      extraDeck.push({ type: 'Xyz', level: null, rank, label: `Xyz Rang ${rank}` });
    });

    const coveredTotals = new Set();
    allCombos.forEach(combo => {
      coveredTotals.add(combo.totalNeeded);
    });

    return {
      extraDeck: extraDeck.slice(0, 15),
      totalCards: extraDeck.length,
      combos: allCombos,
      coveredTotals: [...coveredTotals].sort((a, b) => a - b),
      overflow: extraDeck.length > 15
    };
  });

  const quickTotals = [6, 8, 10, 12, 14, 16, 20];
  const quickTargets = [2, 4, 6, 8, 10, 12];
</script>

<div class="min-h-screen bg-gradient-to-b from-gray-900 to-purple-950 p-3 text-white">
  <div class="max-w-6xl mx-auto">
    
    <!-- Header -->
    <h1 class="text-xl font-bold text-center text-yellow-400 mb-3">
      ∑ SEC Calculator
    </h1>

    <div class="grid md:grid-cols-3 gap-3">
      
      <!-- PANEL 1: Cartes → Cibles -->
      <div class="bg-gray-800/60 rounded-xl p-3">
        <h2 class="text-sm font-bold text-pink-400 mb-2">⇄ Cartes → Cibles</h2>
        
        <div class="flex items-center justify-between mb-1">
          <span class="text-gray-400 text-xs">Total cartes</span>
          <span class="text-xl font-bold text-yellow-400">{totalCards}</span>
        </div>
        <input
          type="range"
          min="3"
          max="23"
          bind:value={totalCards}
          class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-yellow-500"
        />
        
        <div class="flex gap-1 mt-2 flex-wrap">
          {#each quickTotals as n}
            <button
              onclick={() => totalCards = n}
              class="px-2 py-0.5 rounded text-xs transition-all {totalCards === n 
                ? 'bg-yellow-500 text-black font-bold' 
                : 'bg-gray-700 hover:bg-gray-600'}"
            >
              {n}
            </button>
          {/each}
        </div>

        <div class="mt-3 space-y-1 max-h-52 overflow-y-auto pr-1">
          {#if validCombinations().length === 0}
            <div class="text-center text-red-400 py-2 text-xs">∅ Aucune combinaison</div>
          {:else}
            {#each targets() as target}
              <div class="bg-gray-900/50 rounded-lg overflow-hidden">
                <div class="bg-pink-600/80 px-2 py-0.5 text-xs font-bold">◆ Cible {target}</div>
                <div class="p-1.5 space-y-0.5">
                  {#each groupedByTarget()[target] as combo}
                    <div class="flex items-center gap-1 text-xs">
                      <span class="text-yellow-400">F{combo.fusionLevel}</span>
                      <span class="text-gray-500">+</span>
                      <span class="text-cyan-400">★{combo.xyzRank}×2</span>
                    </div>
                  {/each}
                </div>
              </div>
            {/each}
          {/if}
        </div>
      </div>

      <!-- PANEL 2: Cible → Prérequis -->
      <div class="bg-gray-800/60 rounded-xl p-3">
        <h2 class="text-sm font-bold text-cyan-400 mb-2">◇ Cible → Prérequis</h2>
        
        <div class="flex items-center justify-between mb-1">
          <span class="text-gray-400 text-xs">Cible Niv/Rang</span>
          <span class="text-xl font-bold text-cyan-400">{targetLevel}</span>
        </div>
        <input
          type="range"
          min="2"
          max="12"
          bind:value={targetLevel}
          class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-cyan-500"
        />
        
        <div class="flex gap-1 mt-2 flex-wrap">
          {#each quickTargets as n}
            <button
              onclick={() => targetLevel = n}
              class="px-2 py-0.5 rounded text-xs transition-all {targetLevel === n 
                ? 'bg-cyan-500 text-black font-bold' 
                : 'bg-gray-700 hover:bg-gray-600'}"
            >
              {n}
            </button>
          {/each}
        </div>

        <div class="mt-3 space-y-1 max-h-52 overflow-y-auto pr-1">
          {#if prerequisByTarget().length === 0}
            <div class="text-center text-red-400 py-2 text-xs">∅ Aucune combinaison</div>
          {:else}
            {#each prerequisByTarget() as combo}
              <div class="bg-gray-900/50 rounded-lg overflow-hidden">
                <div class="bg-cyan-600/80 px-2 py-0.5 text-xs font-bold">Σ {combo.totalNeeded} cartes</div>
                <div class="p-1.5">
                  <div class="flex items-center gap-1 text-xs">
                    <span class="text-yellow-400">F{combo.fusionLevel}</span>
                    <span class="text-gray-500">+</span>
                    <span class="text-cyan-400">★{combo.xyzRank}×2</span>
                  </div>
                </div>
              </div>
            {/each}
          {/if}
        </div>
      </div>

      <!-- PANEL 3: Extra Deck Builder -->
      <div class="bg-gray-800/60 rounded-xl p-3">
        <h2 class="text-sm font-bold text-green-400 mb-2">✦ Extra Deck Builder</h2>
        
        <!-- Cible 1 -->
        <div class="flex items-center justify-between mb-1">
          <span class="text-gray-400 text-xs">Cible 1</span>
          <span class="text-lg font-bold text-green-400">{target1}</span>
        </div>
        <input
          type="range"
          min="2"
          max="12"
          bind:value={target1}
          class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-green-500"
        />

        <!-- Toggle Cible 2 -->
        <div class="flex items-center gap-2 mt-2 mb-1">
          <button
            onclick={() => useSecondTarget = !useSecondTarget}
            class="px-2 py-0.5 rounded text-xs transition-all {useSecondTarget 
              ? 'bg-green-500 text-black' 
              : 'bg-gray-700'}"
          >
            {useSecondTarget ? '✓' : '+'} Cible 2
          </button>
          {#if useSecondTarget}
            <span class="text-lg font-bold text-green-400">{target2}</span>
          {/if}
        </div>
        {#if useSecondTarget}
          <input
            type="range"
            min="2"
            max="12"
            bind:value={target2}
            class="w-full h-2 bg-gray-700 rounded-lg appearance-none cursor-pointer accent-green-500"
          />
        {/if}

        <!-- Extra Deck généré -->
        <div class="mt-3">
          <div class="text-xs mb-1 {extraDeckBuilder().overflow ? 'text-red-400' : 'text-gray-400'}">
            Extra Deck: {extraDeckBuilder().totalCards}/15 cartes
            {#if extraDeckBuilder().overflow}
              <span class="text-red-400">! Dépasse 15</span>
            {/if}
          </div>
          
          <div class="bg-gray-900/70 rounded-lg p-2 max-h-32 overflow-y-auto">
            <div class="grid grid-cols-2 gap-1">
              {#each extraDeckBuilder().extraDeck as card}
                <div 
                  class="text-xs px-1.5 py-0.5 rounded {card.type === 'Fusion' 
                    ? 'bg-yellow-500/20 text-yellow-400' 
                    : 'bg-cyan-500/20 text-cyan-400'}"
                >
                  {card.type === 'Fusion' ? '◈' : '★'} {card.label}
                </div>
              {/each}
            </div>
          </div>

          <!-- Plages couvertes -->
          <div class="mt-2 text-xs text-gray-400">
            Cartes couvertes: 
            <div class="flex flex-wrap gap-1 mt-1">
              {#each extraDeckBuilder().coveredTotals as n}
                <span class="bg-green-500/20 text-green-400 px-1.5 py-0.5 rounded">
                  {n}
                </span>
              {/each}
            </div>
          </div>
        </div>
      </div>

    </div>

    <!-- Rappel -->
    <div class="mt-3 bg-gray-800/50 rounded-lg p-2 text-xs text-gray-400 text-center">
      <span class="text-yellow-400">Activation :</span> Fusion + 2×Xyz = total cartes → 
      <span class="text-pink-400"> Résolution :</span> Fusion + Xyz = cible adverse
    </div>
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
  }
</style>