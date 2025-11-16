<script lang="ts">
    import pokeballImg from '$lib/assets/pokeball.png';
    import charizard from '$lib/assets/charizard.gif'; 
    import blastoise from '$lib/assets/blastoise.gif';
    import venusaur from '$lib/assets/venusaur.gif'; 
    
    let isAdminMode: boolean = false;
    let familyData: Map<string, string> = new Map(); // Map of name (lowercase) -> family
    let nameOriginalCase: Map<string, string> = new Map(); // Map of name (lowercase) -> original name
    let currentUser: string | null = null;
    let currentFamily: string | null = null;
    let nameInput: string = '';
    let errorMessage: string = '';
    
    // Initialize with test data
    function initializeTestData(): void {
        familyData.set('jayson astor', 'Tight');
        nameOriginalCase.set('jayson astor', 'Jayson Astor');
        familyData.set('kiana sueyoshi', 'Loose');
        nameOriginalCase.set('kiana sueyoshi', 'Kiana Sueyoshi');
    }
    
    // Initialize on mount
    initializeTestData();
    
    function handleCsvUpload(event: Event): void {
        const target = event.target as HTMLInputElement;
        const file = target.files?.[0];
        if (!file) return;
        
        const reader = new FileReader();
        reader.onload = (e: ProgressEvent<FileReader>): void => {
            const csv = e.target?.result;
            if (typeof csv !== 'string') return;
            
            const lines = csv.split('\n');
            
            lines.forEach((line: string) => {
                if (line.trim()) {
                    const [name, family] = line.split(',').map((s: string) => s.trim());
                    if (name && family) {
                        familyData.set(name.toLowerCase(), family);
                        nameOriginalCase.set(name.toLowerCase(), name);
                    }
                }
            });
            
            isAdminMode = false; // Return to main screen after upload
        };
        reader.readAsText(file);
    }
    
    function handleEnter() {
        const family = familyData.get(nameInput.toLowerCase());
        if (family) {
            currentUser = nameOriginalCase.get(nameInput.toLowerCase()) || nameInput;
            currentFamily = family;
            nameInput = '';
            errorMessage = '';
        } else {
            errorMessage = 'Name not found';
        }
    }
    
    function returnToMain() {
        currentUser = null;
        currentFamily = null;
        nameInput = '';
    }
    
    function getScreenColor() {
        if (currentFamily === 'Tight') return 'bg-red-600';
        if (currentFamily === 'Close') return 'bg-green-600';
        if (currentFamily === 'Loose') return 'bg-blue-600';
        return 'bg-black';
    }
    
    function getPokemonImage() {
        if (currentFamily === 'Tight') return charizard;
        if (currentFamily === 'Close') return venusaur;
        if (currentFamily === 'Loose') return blastoise;
        return null;
    }
</script>

<div class="flex items-center justify-center min-h-screen {currentUser ? getScreenColor() : 'bg-black'}">
	<!-- Admin Button -->
	{#if !currentUser}
		<button
			on:click={() => isAdminMode = !isAdminMode}
			class="absolute top-4 right-4 px-4 py-2 bg-gray-700 text-white text-sm font-semibold rounded-lg hover:bg-gray-600 transition-colors"
		>
			{isAdminMode ? 'Back' : 'Admin'}
		</button>
	{/if}

	<div class="w-full max-w-md px-4">
		<div class="text-center">
			{#if currentUser}
				<!-- Result Screen -->
				<h1 class="flex justify-center text-5xl font-bold text-white text-nowrap mb-8">Welcome, {currentUser}!</h1>
				
				<img src={getPokemonImage()} alt="pokemon" class="scale-150 mx-auto my-20" />
				
				<p class="text-5xl font-bold text-white mb-12">To <span>{currentFamily}</span>!</p>
				
				<button
					on:click={returnToMain}
					class="px-6 py-3 bg-white text-{currentFamily === 'Tight' ? 'red' : currentFamily === 'Close' ? 'green' : 'blue'}-600 font-semibold rounded-lg hover:bg-gray-200 transition-colors"
				>
					Return
				</button>
			{:else if isAdminMode}
				<!-- Admin Screen -->
				<h1 class="text-4xl font-bold text-white mb-8">Admin Panel</h1>
				
				<div class="space-y-4">
					<div class="flex items-center justify-center w-full">
						<label for="csv-upload" class="w-full">
							<div class="flex items-center justify-center w-full px-4 py-8 border-2 border-dashed border-gray-400 rounded-lg cursor-pointer hover:border-gray-300 transition-colors">
								<div class="text-center">
									<p class="text-white text-lg font-semibold">Click to upload CSV</p>
									<p class="text-gray-400 text-sm">or drag and drop</p>
								</div>
							</div>
							<input
								id="csv-upload"
								type="file"
								accept=".csv"
								on:change={handleCsvUpload}
								class="hidden"
							/>
						</label>
					</div>
					{#if familyData.size > 0}
						<p class="text-green-400 text-sm">Loaded {familyData.size} entries</p>
					{/if}
				</div>
			{:else}
				<!-- Main Screen -->
				<h1 class="text-4xl font-bold text-white mb-8">What fam are you?</h1>
				
				<img src={pokeballImg} alt="pokeball" class="w-100 h-100 mx-auto mb-8" />
				
				<div class="space-y-4">
					<input
						type="text"
						placeholder="Enter your name"
						bind:value={nameInput}
						on:keydown={(e) => e.key === 'Enter' && handleEnter()}
						class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none"
					>
					<button
						on:click={handleEnter}
						class="w-full px-4 py-2 bg-[#4169E1] text-white font-semibold rounded-lg hover:bg-blue-600 transition-colors"
					>
						Enter
					</button>
					{#if errorMessage}
						<p class="text-red-500 text-sm">{errorMessage}</p>
					{/if}
				</div>
			{/if}
		</div>
	</div>
</div>
