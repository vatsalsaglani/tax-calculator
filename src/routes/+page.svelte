<script lang="ts">
	import DarkCard from '$lib/components/DarkCard.svelte';
	import DarkButton from '$lib/components/DarkButton.svelte';
	import DarkInput from '$lib/components/DarkInput.svelte';
	import DarkLabel from '$lib/components/DarkLabel.svelte';

	let income = '';
	let pf = '';
	let calculatedTax: number | null = null;
	let taxBreakdown: SlabTax[] | null = null;
	let monthlyBeforePF: number | null = null;
	let monthlyAfterPF: number | null = null;

	// Add new variable for tab state
	let activeTab: 'calculator' | 'reforms' = 'calculator';

	// Add new variables for comparison
	let oldRegimeTax2425: number | null = null;
	let newRegimeTax2425: number | null = null;
	let monthlyDifference2425To2526: number | null = null;
	let percentageIncrease2425To2526: number | null = null;

	// Add new variable for result tab state
	let activeResultTab: 'Summary' | 'Breakdown' | 'Detailed Calculation' = 'Summary';

	let oldRegimeBreakdown2425: SlabTax[] = [];
	let newRegimeBreakdown2425: SlabTax[] = [];

	// Add new variables
	let taxReduction2425To2526: number | null = null;
	let taxReductionPercentage: number | null = null;

	interface SlabTax {
		slabRange: string;
		rate: string;
		taxAmount: number;
	}

	function formatCurrency(value: number) {
		return value.toLocaleString('en-IN', {
			style: 'currency',
			currency: 'INR',
			maximumFractionDigits: 0
		});
	}

	function calculateNewRegimeTax2526(totalIncome: number): TaxBreakdown {
		// If totalIncome <= 12,75,000 => 0 tax
		if (totalIncome <= 1275000) {
			return {
				totalTax: 0,
				breakdown: [
					{
						slabRange: 'Up to 12,75,000',
						rate: '0%',
						taxAmount: 0
					}
				]
			};
		}

		const breakdown: SlabTax[] = [];
		let taxableIncome = totalIncome - 75000; // standard deduction
		let totalTax = 0;

		const slabs = [
			{
				slabRange: '0 – 4,00,000',
				limit: 400000,
				rate: 0.0
			},
			{
				slabRange: '4,00,001 – 8,00,000',
				limit: 400000,
				rate: 0.05
			},
			{
				slabRange: '8,00,001 – 12,00,000',
				limit: 400000,
				rate: 0.1
			},
			{
				slabRange: '12,00,001 – 16,00,000',
				limit: 400000,
				rate: 0.15
			},
			{
				slabRange: '16,00,001 – 20,00,000',
				limit: 400000,
				rate: 0.2
			},
			{
				slabRange: '20,00,001 – 24,00,000',
				limit: 400000,
				rate: 0.25
			},
			{
				slabRange: 'Above 24,00,000',
				limit: Infinity,
				rate: 0.3
			}
		];

		for (let i = 0; i < slabs.length; i++) {
			if (taxableIncome <= 0) {
				break;
			}
			const slab = slabs[i];
			if (taxableIncome > slab.limit) {
				const taxed = slab.limit * slab.rate;
				totalTax += taxed;
				breakdown.push({
					slabRange: slab.slabRange,
					rate: `${(slab.rate * 100).toFixed(0)}%`,
					taxAmount: taxed
				});
				taxableIncome -= slab.limit;
			} else {
				const taxed = taxableIncome * slab.rate;
				totalTax += taxed;
				breakdown.push({
					slabRange: slab.slabRange,
					rate: `${(slab.rate * 100).toFixed(0)}%`,
					taxAmount: taxed
				});
				taxableIncome = 0;
				break;
			}
		}

		return {
			totalTax: Math.round(totalTax),
			breakdown: breakdown.map((b) => ({
				...b,
				taxAmount: Math.round(b.taxAmount)
			}))
		};
	}

	function calculateNewRegimeTax2425(totalIncome: number): TaxBreakdown {
		const breakdown: SlabTax[] = [];
		let taxableIncome = totalIncome - 75000; // Updated: FY 24-25 new regime standard deduction is 75,000
		let totalTax = 0;

		const slabs = [
			{
				slabRange: '0 – 3,00,000',
				limit: 300000,
				rate: 0.0
			},
			{
				slabRange: '3,00,001 – 7,00,000',
				limit: 400000,
				rate: 0.05
			},
			{
				slabRange: '7,00,001 – 10,00,000',
				limit: 300000,
				rate: 0.1
			},
			{
				slabRange: '10,00,001 – 12,00,000',
				limit: 200000,
				rate: 0.15
			},
			{
				slabRange: '12,00,001 – 15,00,000',
				limit: 300000,
				rate: 0.2
			},
			{
				slabRange: 'Above 15,00,000',
				limit: Infinity,
				rate: 0.3
			}
		];

		for (let i = 0; i < slabs.length; i++) {
			if (taxableIncome <= 0) break;

			const slab = slabs[i];
			if (taxableIncome > slab.limit) {
				const taxed = slab.limit * slab.rate;
				totalTax += taxed;
				breakdown.push({
					slabRange: slab.slabRange,
					rate: `${(slab.rate * 100).toFixed(0)}%`,
					taxAmount: taxed
				});
				taxableIncome -= slab.limit;
			} else {
				const taxed = taxableIncome * slab.rate;
				totalTax += taxed;
				breakdown.push({
					slabRange: slab.slabRange,
					rate: `${(slab.rate * 100).toFixed(0)}%`,
					taxAmount: taxed
				});
				taxableIncome = 0;
				break;
			}
		}

		return {
			totalTax: Math.round(totalTax),
			breakdown: breakdown.map((b) => ({
				...b,
				taxAmount: Math.round(b.taxAmount)
			}))
		};
	}

	function calculateOldRegimeTax2425(totalIncome: number): TaxBreakdown {
		const breakdown: SlabTax[] = [];
		let taxableIncome = totalIncome - 50000; // Updated: FY 24-25 old regime standard deduction is 50,000
		let totalTax = 0;

		const slabs = [
			{
				slabRange: '0 – 2,50,000',
				limit: 250000,
				rate: 0.0
			},
			{
				slabRange: '2,50,001 – 5,00,000',
				limit: 250000,
				rate: 0.05
			},
			{
				slabRange: '5,00,001 – 10,00,000',
				limit: 500000,
				rate: 0.2
			},
			{
				slabRange: 'Above 10,00,000',
				limit: Infinity,
				rate: 0.3
			}
		];

		for (let i = 0; i < slabs.length; i++) {
			if (taxableIncome <= 0) break;

			const slab = slabs[i];
			if (taxableIncome > slab.limit) {
				const taxed = slab.limit * slab.rate;
				totalTax += taxed;
				breakdown.push({
					slabRange: slab.slabRange,
					rate: `${(slab.rate * 100).toFixed(0)}%`,
					taxAmount: taxed
				});
				taxableIncome -= slab.limit;
			} else {
				const taxed = taxableIncome * slab.rate;
				totalTax += taxed;
				breakdown.push({
					slabRange: slab.slabRange,
					rate: `${(slab.rate * 100).toFixed(0)}%`,
					taxAmount: taxed
				});
				taxableIncome = 0;
				break;
			}
		}

		return {
			totalTax: Math.round(totalTax),
			breakdown: breakdown.map((b) => ({
				...b,
				taxAmount: Math.round(b.taxAmount)
			}))
		};
	}

	function handleCalculate() {
		const totalIncome = parseFloat(income || '0');
		if (isNaN(totalIncome) || totalIncome <= 0) {
			// Reset all state variables
			calculatedTax = null;
			taxBreakdown = null;
			monthlyBeforePF = null;
			monthlyAfterPF = null;
			oldRegimeTax2425 = null;
			newRegimeTax2425 = null;
			monthlyDifference2425To2526 = null;
			percentageIncrease2425To2526 = null;
			oldRegimeBreakdown2425 = [];
			newRegimeBreakdown2425 = [];
			return;
		}

		// Calculate FY 25-26 tax
		const { totalTax: tax2526, breakdown: breakdown2526 } = calculateNewRegimeTax2526(totalIncome);
		const monthlyTakeHome2526 = (totalIncome - tax2526) / 12;

		// Calculate FY 24-25 taxes for both regimes
		const { totalTax: newTax2425 } = calculateNewRegimeTax2425(totalIncome);
		const { totalTax: oldTax2425 } = calculateOldRegimeTax2425(totalIncome);

		// Use the better regime's take-home (lower tax) as base for comparison
		const bestTax2425 = Math.min(newTax2425, oldTax2425);
		const monthlyTakeHome2425 = (totalIncome - bestTax2425) / 12;

		// Calculate differences
		monthlyDifference2425To2526 = monthlyTakeHome2526 - monthlyTakeHome2425;
		percentageIncrease2425To2526 = (monthlyDifference2425To2526 / monthlyTakeHome2425) * 100;

		// Calculate tax reduction
		const taxReduction = bestTax2425 - tax2526;
		taxReduction2425To2526 = taxReduction;
		taxReductionPercentage = (taxReduction / bestTax2425) * 100;

		// Set all state variables
		calculatedTax = tax2526;
		oldRegimeTax2425 = oldTax2425;
		newRegimeTax2425 = newTax2425;
		monthlyBeforePF = monthlyTakeHome2526;

		// Calculate after PF if provided
		const monthlyPfFloat = parseFloat(pf || '0');
		monthlyAfterPF =
			!isNaN(monthlyPfFloat) && monthlyPfFloat > 0
				? monthlyTakeHome2526 - monthlyPfFloat * 2
				: null;

		// Store breakdowns for detailed view
		const oldRegimeResult = calculateOldRegimeTax2425(totalIncome);
		const newRegimeResult = calculateNewRegimeTax2425(totalIncome);

		oldRegimeBreakdown2425 = oldRegimeResult.breakdown;
		newRegimeBreakdown2425 = newRegimeResult.breakdown;

		// Store the breakdown
		taxBreakdown = breakdown2526;
	}
</script>

<div class="bg-gradient-to-tr from-gray-900 via-gray-800 to-black min-h-screen text-gray-100">
	<!-- Hero Section -->
	<div class="p-4 md:p-8 text-center">
		<h1 class="text-2xl md:text-3xl font-bold mb-2">Union Budget 2025–26</h1>
		<p class="text-base md:text-lg font-semibold mb-2">
			Zero Income Tax till ₹12 Lakh under New Tax Regime
		</p>
		<p class="text-sm md:text-md">
			(₹12.75 lakh for salaried taxpayers with a ₹75,000 standard deduction)
		</p>
	</div>

	<!-- Mobile Tabs -->
	<div class="md:hidden px-4 mb-4">
		<div class="flex rounded-lg bg-gray-800 p-1">
			<button
				class="flex-1 py-2 px-4 rounded-md text-sm font-medium transition-colors {activeTab ===
				'calculator'
					? 'bg-blue-600 text-white'
					: 'text-gray-300 hover:text-white'}"
				on:click={() => (activeTab = 'calculator')}
			>
				Calculator
			</button>
			<button
				class="flex-1 py-2 px-4 rounded-md text-sm font-medium transition-colors {activeTab ===
				'reforms'
					? 'bg-blue-600 text-white'
					: 'text-gray-300 hover:text-white'}"
				on:click={() => (activeTab = 'reforms')}
			>
				Tax Reforms
			</button>
		</div>
	</div>

	<div class="container mx-auto px-4 pb-8 flex flex-col md:flex-row gap-8">
		<!-- Left: Reforms Table (hidden on mobile when calculator tab is active) -->
		<div class="md:w-1/3 w-full {activeTab === 'calculator' ? 'hidden md:block' : ''}">
			<DarkCard>
				<h2 class="text-xl font-semibold mb-4">
					Personal Income Tax Reforms with Special Focus on Middle Class
				</h2>
				<ul class="list-disc list-inside space-y-2 mb-6">
					<li>
						<strong>'Nil tax' slab</strong> up to ₹12.00 lakh (effectively ₹12.75 lakh for salaried taxpayers
						with the ₹75,000 standard deduction).
					</li>
					<li>
						New structure to substantially reduce taxes of middle class and leave more money in
						their hands, boosting consumption, savings, and investment.
					</li>
					<li>Slabs and rates have been changed across the board to benefit all taxpayers.</li>
				</ul>
				<div class="overflow-x-auto mb-6">
					<table class="w-full text-left border-collapse text-sm">
						<thead>
							<tr>
								<th class="px-3 py-2 border border-gray-700 bg-gray-700">Income (₹ Lakhs)</th>
								<th class="px-3 py-2 border border-gray-700 bg-gray-700">Tax Rate</th>
							</tr>
						</thead>
						<tbody>
							<tr>
								<td class="px-3 py-2 border border-gray-700">0 – 4</td>
								<td class="px-3 py-2 border border-gray-700">Nil</td>
							</tr>
							<tr>
								<td class="px-3 py-2 border border-gray-700">4 – 8</td>
								<td class="px-3 py-2 border border-gray-700">5%</td>
							</tr>
							<tr>
								<td class="px-3 py-2 border border-gray-700">8 – 12</td>
								<td class="px-3 py-2 border border-gray-700">10%</td>
							</tr>
							<tr>
								<td class="px-3 py-2 border border-gray-700">12 – 16</td>
								<td class="px-3 py-2 border border-gray-700">15%</td>
							</tr>
							<tr>
								<td class="px-3 py-2 border border-gray-700">16 – 20</td>
								<td class="px-3 py-2 border border-gray-700">20%</td>
							</tr>
							<tr>
								<td class="px-3 py-2 border border-gray-700">20 – 24</td>
								<td class="px-3 py-2 border border-gray-700">25%</td>
							</tr>
							<tr>
								<td class="px-3 py-2 border border-gray-700">Above 24</td>
								<td class="px-3 py-2 border border-gray-700">30%</td>
							</tr>
						</tbody>
					</table>
				</div>
				<p class="text-xs text-gray-400">
					<strong>Disclaimer:</strong> The above slabs are per the new regime as illustrated in the Union
					Budget 2025–26. Real-world scenarios may include surcharges & cesses.
				</p>
			</DarkCard>
		</div>

		<!-- Right: Calculator (hidden on mobile when reforms tab is active) -->
		<div class="md:w-2/3 w-full {activeTab === 'reforms' ? 'hidden md:block' : ''}">
			<DarkCard>
				<h2 class="text-xl font-semibold mb-4">New Tax Regime Calculator</h2>

				<form on:submit|preventDefault={handleCalculate}>
					<div class="space-y-4">
						<div>
							<DarkLabel for="income">Annual Income (₹):</DarkLabel>
							<DarkInput
								type="text"
								id="income"
								bind:value={income}
								placeholder="Enter your annual income"
							/>
						</div>

						<div>
							<DarkLabel for="pf">Monthly PF (Employee portion) (₹):</DarkLabel>
							<DarkInput
								type="text"
								id="pf"
								bind:value={pf}
								placeholder="Optional: Enter your monthly PF contribution"
							/>
						</div>

						<DarkButton type="submit">Calculate</DarkButton>
					</div>
				</form>

				<!-- Results Section -->
				{#if calculatedTax !== null}
					<div class="mt-6">
						<!-- Tabs -->
						<div class="border-b border-gray-700 mb-6">
							<div class="overflow-x-auto -mb-px" style="scrollbar-width: none;">
								<nav class="flex min-w-max" aria-label="Tabs">
									{#each ['Summary', 'Breakdown', 'Detailed Calculation'] as tab}
										<button
											class="whitespace-nowrap py-2 px-4 border-b-2 font-medium text-sm cursor-pointer transition-colors
												{activeResultTab === tab
												? 'border-blue-500 text-blue-400'
												: 'border-transparent text-gray-400 hover:text-gray-300 hover:border-gray-300'}"
											on:click={() => (activeResultTab = tab)}
										>
											{tab}
										</button>
									{/each}
								</nav>
							</div>
						</div>

						<!-- Summary Tab -->
						{#if activeResultTab === 'Summary'}
							<!-- Monthly Take-Home Cards -->
							<div
								class="grid grid-cols-1 {monthlyAfterPF !== null
									? 'md:grid-cols-2'
									: ''} gap-4 mb-6"
							>
								<div class="bg-gray-900/50 rounded-lg p-6">
									<p class="text-gray-400 mb-2">Monthly Take-Home (Before PF)</p>
									<div class="flex items-baseline">
										<h4 class="text-2xl font-bold text-white">
											{formatCurrency(Math.round(monthlyBeforePF))}
										</h4>
										{#if monthlyDifference2425To2526}
											<span class="ml-3 text-green-400 text-lg">
												₹{Math.abs(Math.round(monthlyDifference2425To2526)).toLocaleString('en-IN')}
											</span>
										{/if}
									</div>
									<p class="text-sm text-gray-400 mt-2">
										Based on taxable income of {formatCurrency(parseFloat(income) - 75000)} (after standard
										deduction of ₹75,000)
									</p>
								</div>

								{#if monthlyAfterPF !== null}
									<div class="bg-gray-900/50 rounded-lg p-6">
										<p class="text-gray-400 mb-2">Monthly Take-Home (After PF)</p>
										<div class="flex items-baseline">
											<h4 class="text-2xl font-bold text-white">
												{monthlyAfterPF < 0 ? 'N/A' : formatCurrency(Math.round(monthlyAfterPF))}
											</h4>
											{#if monthlyDifference2425To2526}
												<span class="ml-3 text-green-400 text-lg">
													₹{Math.abs(
														Math.round(monthlyDifference2425To2526 - (parseFloat(pf) * 2 || 0))
													).toLocaleString('en-IN')}
												</span>
											{/if}
										</div>
										<p class="text-sm text-gray-400 mt-2">
											Calculated as monthly take-home minus total PF contribution ({formatCurrency(
												parseFloat(pf) * 2
											)}), which includes both employee ({formatCurrency(parseFloat(pf))}) and
											employer ({formatCurrency(parseFloat(pf))}) contributions
										</p>
									</div>
								{/if}
							</div>

							<!-- Tax Regime Comparison -->
							<div class="bg-blue-900/30 border border-blue-700 rounded-lg p-6">
								<h4 class="text-xl font-semibold text-white mb-4">Tax Regime Comparison</h4>

								<!-- FY 2024-25 -->
								<div class="mb-6">
									<h5 class="text-lg text-gray-200 mb-2">FY 2024-25</h5>
									<div class="space-y-2">
										<div class="flex justify-between">
											<span class="text-gray-300">New Regime Tax:</span>
											<span class="text-white"
												>₹{Math.round(newRegimeTax2425).toLocaleString('en-IN')}</span
											>
										</div>
										<div class="flex justify-between">
											<span class="text-gray-300">Old Regime Tax:</span>
											<span class="text-white"
												>₹{Math.round(oldRegimeTax2425).toLocaleString('en-IN')}</span
											>
										</div>
										<div class="flex justify-between pt-2 border-t border-blue-700">
											<span class="text-gray-300">Best Option Savings:</span>
											<span class="text-green-400"
												>₹{Math.abs(Math.round(newRegimeTax2425 - oldRegimeTax2425)).toLocaleString(
													'en-IN'
												)}</span
											>
										</div>
									</div>
								</div>

								<!-- FY 2025-26 -->
								<div class="mb-6">
									<h5 class="text-lg text-gray-200 mb-2">FY 2025-26 (New Regime)</h5>
									<div class="flex justify-between">
										<span class="text-gray-300">Tax Amount:</span>
										<span class="text-white"
											>₹{Math.round(calculatedTax).toLocaleString('en-IN')}</span
										>
									</div>
								</div>

								<!-- Year-over-Year Change -->
								<div class="pt-4 border-t border-blue-700">
									<h5 class="text-lg text-gray-200 mb-2">Year-over-Year Change</h5>
									<div class="space-y-2">
										<div class="flex justify-between">
											<span class="text-gray-300">Monthly Take-Home Increase:</span>
											<span class="text-green-400"
												>+₹{Math.round(monthlyDifference2425To2526).toLocaleString('en-IN')}</span
											>
										</div>
										<div class="flex justify-between">
											<span class="text-gray-300">Percentage Change:</span>
											<span class="text-green-400">+{percentageIncrease2425To2526.toFixed(2)}%</span
											>
										</div>
										<!-- Add Tax Reduction Information -->
										<div class="flex justify-between">
											<span class="text-gray-300">Annual Tax Reduction:</span>
											<span class="text-green-400"
												>₹{Math.round(taxReduction2425To2526).toLocaleString('en-IN')}</span
											>
										</div>
										<div class="flex justify-between">
											<span class="text-gray-300">Tax Reduction %:</span>
											<span class="text-green-400">{taxReductionPercentage.toFixed(2)}%</span>
										</div>
									</div>
								</div>
							</div>
						{/if}

						<!-- Breakdown Tab -->
						{#if activeResultTab === 'Breakdown'}
							<div class="overflow-x-auto">
								<table class="w-full text-left border-collapse">
									<thead>
										<tr>
											<th class="px-4 py-3 bg-gray-800 text-gray-200 border border-gray-700"
												>Slab Range</th
											>
											<th class="px-4 py-3 bg-gray-800 text-gray-200 border border-gray-700"
												>Rate</th
											>
											<th class="px-4 py-3 bg-gray-800 text-gray-200 border border-gray-700"
												>Tax Amount</th
											>
										</tr>
									</thead>
									<tbody>
										{#if taxBreakdown && taxBreakdown.length > 0}
											{#each taxBreakdown as item}
												<tr class="border-t border-gray-700">
													<td class="px-4 py-3 border border-gray-700">{item.slabRange}</td>
													<td class="px-4 py-3 border border-gray-700">{item.rate}</td>
													<td class="px-4 py-3 border border-gray-700"
														>{formatCurrency(item.taxAmount)}</td
													>
												</tr>
											{/each}
											<!-- Total Row -->
											<tr class="border-t border-gray-700 bg-gray-800">
												<td class="px-4 py-3 border border-gray-700 font-medium" colspan="2"
													>Total Tax</td
												>
												<td class="px-4 py-3 border border-gray-700 font-medium"
													>{formatCurrency(calculatedTax)}</td
												>
											</tr>
										{/if}
									</tbody>
								</table>
							</div>
						{/if}

						{#if activeResultTab === 'Detailed Calculation'}
							<div class="space-y-6">
								<!-- FY 2024-25 Old Regime -->
								<div class="bg-gray-900/50 rounded-lg p-4">
									<h3 class="text-lg font-semibold mb-3">FY 2024-25 Old Regime Calculation</h3>
									<div class="space-y-2">
										<p>Gross Income: {formatCurrency(parseFloat(income))}</p>
										<p>Standard Deduction: {formatCurrency(50000)}</p>
										<div class="mt-4">
											<h4 class="font-medium mb-2">Slab-wise Calculation:</h4>
											{#each oldRegimeBreakdown2425 as item}
												<div class="pl-4 mb-2">
													<p>{item.slabRange} @ {item.rate}</p>
													<p class="text-gray-400">Tax: {formatCurrency(item.taxAmount)}</p>
												</div>
											{/each}
										</div>
										<p class="font-medium mt-3">Total Tax: {formatCurrency(oldRegimeTax2425)}</p>
									</div>
								</div>

								<!-- FY 2024-25 New Regime -->
								<div class="bg-gray-900/50 rounded-lg p-4">
									<h3 class="text-lg font-semibold mb-3">FY 2024-25 New Regime Calculation</h3>
									<div class="space-y-2">
										<p>Gross Income: {formatCurrency(parseFloat(income))}</p>
										<p>Standard Deduction: {formatCurrency(75000)}</p>
										<div class="mt-4">
											<h4 class="font-medium mb-2">Slab-wise Calculation:</h4>
											{#each newRegimeBreakdown2425 as item}
												<div class="pl-4 mb-2">
													<p>{item.slabRange} @ {item.rate}</p>
													<p class="text-gray-400">Tax: {formatCurrency(item.taxAmount)}</p>
												</div>
											{/each}
										</div>
										<p class="font-medium mt-3">Total Tax: {formatCurrency(newRegimeTax2425)}</p>
									</div>
								</div>

								<!-- FY 2025-26 New Regime -->
								<div class="bg-gray-900/50 rounded-lg p-4">
									<h3 class="text-lg font-semibold mb-3">FY 2025-26 New Regime Calculation</h3>
									<div class="space-y-2">
										<p>Gross Income: {formatCurrency(parseFloat(income))}</p>
										<p>Standard Deduction: {formatCurrency(75000)}</p>
										<div class="mt-4">
											<h4 class="font-medium mb-2">Slab-wise Calculation:</h4>
											{#each taxBreakdown as item}
												<div class="pl-4 mb-2">
													<p>{item.slabRange} @ {item.rate}</p>
													<p class="text-gray-400">Tax: {formatCurrency(item.taxAmount)}</p>
												</div>
											{/each}
										</div>
										<p class="font-medium mt-3">Total Tax: {formatCurrency(calculatedTax)}</p>
									</div>
								</div>
							</div>
						{/if}
					</div>
				{/if}
			</DarkCard>
		</div>
	</div>

	<!-- Add this at the bottom of your page, before closing the main div -->
	<footer class="w-full py-6 mt-auto border-t border-[#1E3E62]">
		<div class="container mx-auto px-4">
			<div class="flex flex-col md:flex-row items-center justify-center gap-2 text-sm text-gray-400">
				<span>Developed by</span>
				<div class="flex items-center gap-2 flex-wrap justify-center">
					<a 
						href="https://x.com/@saglanivatsal" 
						target="_blank" 
						rel="noopener noreferrer"
						class="text-[#FF6500] hover:text-[#FF8500] transition-colors flex items-center gap-1"
					>
						<svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
							<path d="M18.244 2.25h3.308l-7.227 8.26 8.502 11.24H16.17l-5.214-6.817L4.99 21.75H1.68l7.73-8.835L1.254 2.25H8.08l4.713 6.231zm-1.161 17.52h1.833L7.084 4.126H5.117z"/>
						</svg>
						<span>@saglanivatsal</span>
					</a>
					<span class="text-gray-500">•</span>
					<a 
						href="https://thevatsalsaglani.medium.com" 
						target="_blank" 
						rel="noopener noreferrer"
						class="text-[#FF6500] hover:text-[#FF8500] transition-colors flex items-center gap-1"
					>
						<svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
							<path d="M13.54 12a6.8 6.8 0 01-6.77 6.82A6.8 6.8 0 010 12a6.8 6.8 0 016.77-6.82A6.8 6.8 0 0113.54 12zM20.96 12c0 3.54-1.51 6.42-3.38 6.42-1.87 0-3.39-2.88-3.39-6.42s1.52-6.42 3.39-6.42 3.38 2.88 3.38 6.42M24 12c0 3.17-.53 5.75-1.19 5.75-.66 0-1.19-2.58-1.19-5.75s.53-5.75 1.19-5.75C23.47 6.25 24 8.83 24 12z"/>
						</svg>
						<span>Medium</span>
					</a>
				</div>
				<div class="flex items-center gap-2 flex-wrap justify-center">
					<span class="text-gray-400">using</span>
					<a 
						href="https://chatgpt.com"
						target="_blank" 
						rel="noopener noreferrer"
						class="text-[#FF6500] hover:text-[#FF8500] transition-colors"
					>OpenAI o1</a>
					<span class="text-gray-500">+</span>
					<a 
						href="https://www.cursor.com"
						target="_blank" 
						rel="noopener noreferrer"
						class="text-[#FF6500] hover:text-[#FF8500] transition-colors"
					>Cursor</a>
					<span class="text-gray-500">+</span>
					<a 
						href="https://claude.ai"
						target="_blank" 
						rel="noopener noreferrer"
						class="text-[#FF6500] hover:text-[#FF8500] transition-colors"
					>Claude 3.5 Sonnet</a>
				</div>
			</div>
		</div>
	</footer>
</div>

<style>
	/* Hide scrollbar for Chrome, Safari and Opera */
	.overflow-x-auto::-webkit-scrollbar {
		display: none;
	}

	/* Hide scrollbar for IE, Edge and Firefox */
	.overflow-x-auto {
		-ms-overflow-style: none; /* IE and Edge */
		scrollbar-width: none; /* Firefox */
	}

	/* Make sure the footer sticks to the bottom */
	.min-h-screen {
		display: flex;
		flex-direction: column;
	}
</style>
