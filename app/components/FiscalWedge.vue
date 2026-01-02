<script setup>
    import { ref, computed } from "vue";

    /* =======================
   PROPS (Scenario Inputs)
======================= */
    const props = defineProps({
        baseRoyalty: {
            type: Number,
            default: 0.05,
        },

        aiscMode: {
            type: String,
            default: "fixed", // 'fixed' | 'variable'
        },

        fixedAisc: {
            type: Number,
            default: 610,
        },

        variableAisc: {
            type: Number,
            default: 645.7,
        },

        showSmoothingToggle: {
            type: Boolean,
            default: true,
        },

        price: {
            type: Number,
            default: 3050,
        },

        volume: {
            type: Number,
            default: 500000,
        },
    });

    /* =======================
   LOCAL STATE (User-controlled)
======================= */
    const price = ref(props.price);
    const volume = ref(props.volume);
    const useSmoothing = ref(false);

    /* =======================
   CONSTANTS
======================= */
    const GSL_RATE = 0.01;
    const CIT_RATE = 0.35;
    const GOV_EQUITY = 0.19;
    const COMP_EQUITY = 0.81;

    /* =======================
   EFFECTIVE AISC (KEY FIX)
======================= */
    const effectiveAisc = computed(() => {
        // FIXED COST SCENARIOS (A / B)
        if (props.aiscMode === "fixed") {
            return props.fixedAisc;
        }

        // VARIABLE COST SCENARIOS (C / D)
        if (props.aiscMode === "variable") {
            if (volume.value < 300_000) return 700;
            if (volume.value < 1_000_000) return 645.7;
            return 610;
        }

        return 610;
    });

    /* =======================
   ROYALTY ALGORITHMS
======================= */
    const getSmoothedRate = (price) => {
        if (price <= 1500) return 0.05;
        if (price <= 2500) return 0.05 + ((price - 1500) / 1000) * 0.02;
        if (price <= 3000) return 0.07 + ((price - 2500) / 500) * 0.03;
        if (price <= 4000) return 0.1 + ((price - 3000) / 1000) * 0.02;
        return 0.12;
    };

    const getStepRate = (price) => {
        if (price <= 1500) return 0.05;
        if (price <= 2500) return 0.07;
        if (price <= 3000) return 0.1;
        return 0.12;
    };

    /* =======================
   CORE CALCULATION
======================= */
    const calculateSplit = (lithiumPrice, aisc) => {
        const royaltyRate = useSmoothing.value
            ? getSmoothedRate(lithiumPrice)
            : getStepRate(lithiumPrice);

        const royalty = lithiumPrice * royaltyRate;
        const gsl = lithiumPrice * GSL_RATE;

        const taxableIncome = Math.max(0, lithiumPrice - aisc - royalty);

        const corpTax = taxableIncome * CIT_RATE;
        const postTaxProfit = taxableIncome - corpTax - gsl;

        const distributable = Math.max(0, postTaxProfit);
        const govDividend = distributable * GOV_EQUITY;
        const compDividend = distributable * COMP_EQUITY;

        const govTotal = royalty + gsl + corpTax + govDividend;
        const compTotal = postTaxProfit > 0 ? compDividend : postTaxProfit;

        const totalPie = Math.max(0, govTotal) + Math.max(0, compTotal);

        return {
            govTotal,
            compTotal,
            govShare: totalPie > 0 ? (govTotal / totalPie) * 100 : 0,
            compShare: totalPie > 0 ? (compTotal / totalPie) * 100 : 0,
            royaltyRate,
        };
    };

    /* =======================
   RESULTS (Single Scenario)
======================= */
    const results = computed(() => {
        const res = calculateSplit(price.value, effectiveAisc.value);

        // scale to annual values
        res.govTotal *= volume.value;
        res.compTotal *= volume.value;

        return [
            {
                id: 1,
                title:
                    props.aiscMode === "fixed"
                        ? "Fixed Costs"
                        : "Variable Costs",
                desc:
                    props.aiscMode === "fixed"
                        ? `AISC $${effectiveAisc.value.toFixed(0)}`
                        : `Volume-dependent AISC ($${effectiveAisc.value.toFixed(
                              1
                          )})`,
                data: res,
            },
        ];
    });

    /* =======================
   CLIFF EFFECT ANALYSIS
======================= */
    const cliffAnalysis = computed(() => {
        const thresholds = [
            { limit: 1500, low: 0.05, high: 0.07 },
            { limit: 2500, low: 0.07, high: 0.1 },
            { limit: 3000, low: 0.1, high: 0.12 },
        ];

        const p = price.value;
        const near = thresholds.find((t) => p > t.limit && p <= t.limit + 150);

        if (!near) return { isNear: false };

        const gamingPrice = near.limit;
        const gamingResults = calculateSplit(gamingPrice, effectiveAisc.value);

        return {
            isNear: true,
            threshold: near.limit,
            lowRate: near.low,
            highRate: near.high,
            gamingProfit: gamingResults.compTotal * volume.value,
        };
    });

    /* =======================
   FORMATTERS
======================= */
    const formatNum = (v) => new Intl.NumberFormat("en-US").format(v);

    const formatMillions = (val) =>
        "$" +
        (val / 1_000_000).toLocaleString("en-US", {
            minimumFractionDigits: 1,
            maximumFractionDigits: 1,
        }) +
        "M";

    const formatPct = (val) =>
        new Intl.NumberFormat("en-US", {
            style: "percent",
            minimumFractionDigits: 1,
        }).format(val);
</script>

<template>
    <div
        class="max-w-5xl mx-auto p-6 bg-gray-50 rounded-xl font-sans text-gray-800"
    >
        <!-- Header -->
        <div class="mb-8 text-center">
            <h2 class="text-3xl font-extrabold text-gray-900 mb-3">
                Lithium Royalty: The "Cliff Effect"
            </h2>
            <p class="text-gray-600 max-w-2xl mx-auto">
                Visualize how fixed royalty thresholds create perverse
                incentives to "game" the price. Compare the
                <strong>Step (Cliff)</strong> model vs. a
                <strong>Smoothed</strong> model.
            </p>
        </div>

        <!-- Main Control Panel -->
        <div
            class="bg-white rounded-xl shadow-lg border border-gray-200 overflow-hidden mb-8"
        >
            <div class="p-6 md:p-8 bg-white">
                <!-- Top Row: Volume & Algorithm -->
                <div class="flex flex-col md:flex-row gap-8 mb-8">
                    <!-- Volume Input -->
                    <div class="flex-1">
                        <label
                            class="block text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide"
                        >
                            Annual Production (Tonnes)
                        </label>
                        <div class="relative rounded-md shadow-sm">
                            <input
                                type="number"
                                v-model.number="volume"
                                class="block w-full text-lg border-gray-300 rounded-lg pl-4 pr-12 py-3 bg-gray-50 focus:ring-blue-500 focus:border-blue-500 border"
                            />
                            <div
                                class="absolute inset-y-0 right-0 pr-4 flex items-center pointer-events-none"
                            >
                                <span class="text-gray-500 font-medium">t</span>
                            </div>
                        </div>
                        <p class="text-xs text-gray-400 mt-2">
                            Adjust to see impact on annual revenues (e.g.,
                            500,000t).
                        </p>
                    </div>

                    <!-- Algorithm Toggle -->
                    <div class="flex-1">
                        <label
                            class="block text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide"
                        >
                            Royalty Regime
                        </label>
                        <div class="flex bg-gray-100 p-1 rounded-lg">
                            <button
                                @click="useSmoothing = false"
                                class="flex-1 py-3 px-4 rounded-md text-sm font-bold transition-all duration-200 flex items-center justify-center gap-2"
                                :class="
                                    !useSmoothing
                                        ? 'bg-white text-red-600 shadow-md ring-1 ring-black/5'
                                        : 'text-gray-500 hover:text-gray-700'
                                "
                            >
                                <!-- Icon: Bar Chart -->
                                <svg
                                    xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)"
                                    width="16"
                                    height="16"
                                    viewBox="0 0 24 24"
                                    fill="none"
                                    stroke="currentColor"
                                    stroke-width="2"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                >
                                    <line
                                        x1="18"
                                        y1="20"
                                        x2="18"
                                        y2="10"
                                    ></line>
                                    <line x1="12" y1="20" x2="12" y2="4"></line>
                                    <line x1="6" y1="20" x2="6" y2="14"></line>
                                </svg>
                                Step (Cliff)
                            </button>
                            <button
                                @click="useSmoothing = true"
                                class="flex-1 py-3 px-4 rounded-md text-sm font-bold transition-all duration-200 flex items-center justify-center gap-2"
                                :class="
                                    useSmoothing
                                        ? 'bg-white text-green-600 shadow-md ring-1 ring-black/5'
                                        : 'text-gray-500 hover:text-gray-700'
                                "
                            >
                                <!-- Icon: Activity -->
                                <svg
                                    xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)"
                                    width="16"
                                    height="16"
                                    viewBox="0 0 24 24"
                                    fill="none"
                                    stroke="currentColor"
                                    stroke-width="2"
                                    stroke-linecap="round"
                                    stroke-linejoin="round"
                                >
                                    <polyline
                                        points="22 12 18 12 15 21 9 3 6 12 2 12"
                                    ></polyline>
                                </svg>
                                Smoothed
                            </button>
                        </div>
                        <p
                            class="text-xs text-gray-400 mt-2 text-center"
                            v-if="!useSmoothing"
                        >
                            Current Ghana Model (Fixed Bands)
                        </p>
                        <p
                            class="text-xs text-gray-400 mt-2 text-center"
                            v-else
                        >
                            Proposed Linear Interpolation
                        </p>
                    </div>
                </div>

                <!-- Price Slider -->
                <div class="mb-2">
                    <div class="flex justify-between items-end mb-4">
                        <label
                            class="text-sm font-bold text-gray-700 uppercase tracking-wide"
                            >Market Price (USD/t)</label
                        >
                        <div
                            class="text-3xl font-black text-blue-600 tracking-tight"
                        >
                            ${{ formatNum(price) }}
                        </div>
                    </div>
                    <input
                        type="range"
                        min="1000"
                        max="4000"
                        step="10"
                        v-model.number="price"
                        class="w-full h-4 bg-gray-200 rounded-full appearance-none cursor-pointer accent-blue-600 hover:accent-blue-700 transition-all"
                    />
                    <div
                        class="flex justify-between text-xs font-medium text-gray-400 mt-2"
                    >
                        <span>$1,000</span>
                        <span class="pl-8">$2,500 (7%→10%)</span>
                        <span class="pl-8">$3,000 (10%→12%)</span>
                        <span>$4,000</span>
                    </div>
                </div>
            </div>

            <!-- CLIFF ANALYZER (Only shows in Step mode near thresholds) -->
            <div
                v-if="!useSmoothing && cliffAnalysis.isNear"
                class="border-t border-red-100 bg-red-50/50 p-6 md:p-8 rounded-b-xl mb-8"
            >
                <div class="flex items-start gap-4">
                    <div class="bg-red-100 p-3 rounded-full hidden md:block">
                        <!-- Icon: Alert Triangle -->
                        <svg
                            class="w-6 h-6 text-red-600"
                            fill="none"
                            stroke="currentColor"
                            viewBox="0 0 24 24"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path
                                d="M10.29 3.86L1.82 18a2 2 0 0 0 1.71 3h16.94a2 2 0 0 0 1.71-3L13.71 3.86a2 2 0 0 0-3.42 0z"
                            ></path>
                            <line x1="12" y1="9" x2="12" y2="13"></line>
                            <line x1="12" y1="17" x2="12.01" y2="17"></line>
                        </svg>
                    </div>
                    <div class="flex-1">
                        <h3 class="text-lg font-bold text-red-800 mb-1">
                            ⚠️ The Cliff Effect Detected
                        </h3>
                        <p class="text-red-700 mb-4 text-sm leading-relaxed">
                            At the current price of <strong>${{ price }}</strong
                            >, the royalty rate is
                            <strong
                                >{{
                                    (cliffAnalysis.highRate * 100).toFixed(0)
                                }}%</strong
                            >. By artificially lowering the price to
                            <strong>${{ cliffAnalysis.threshold }}</strong
                            >, the rate drops to
                            <strong
                                >{{
                                    (cliffAnalysis.lowRate * 100).toFixed(0)
                                }}%</strong
                            >. <br />See how "earning less" actually creates
                            <strong>more profit</strong> for the company:
                        </p>

                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                            <!-- High Price Scenario (Current) -->
                            <div
                                class="bg-white p-4 rounded-lg border border-red-200 shadow-sm opacity-75"
                            >
                                <div
                                    class="text-xs font-bold text-red-500 uppercase mb-2"
                                >
                                    Scenario A: Honest Pricing
                                </div>
                                <div class="flex justify-between mb-1">
                                    <span class="text-sm text-gray-600"
                                        >Price:</span
                                    >
                                    <span class="font-bold text-gray-800"
                                        >${{ price }}</span
                                    >
                                </div>
                                <div class="flex justify-between mb-1">
                                    <span class="text-sm text-gray-600"
                                        >Revenue:</span
                                    >
                                    <span class="font-medium text-gray-800">{{
                                        formatMillions(price * volume)
                                    }}</span>
                                </div>
                                <div class="flex justify-between mb-1">
                                    <span class="text-sm text-gray-600"
                                        >Royalty ({{
                                            (
                                                cliffAnalysis.highRate * 100
                                            ).toFixed(0)
                                        }}%):</span
                                    >
                                    <span class="font-bold text-red-600">{{
                                        formatMillions(
                                            price *
                                                volume *
                                                cliffAnalysis.highRate
                                        )
                                    }}</span>
                                </div>
                                <div
                                    class="mt-2 pt-2 border-t border-gray-100 flex justify-between"
                                >
                                    <span
                                        class="text-sm font-bold text-gray-700"
                                        >Net Profit:</span
                                    >
                                    <span class="font-bold text-gray-900">{{
                                        formatMillions(
                                            results[0].data.compTotal
                                        )
                                    }}</span>
                                </div>
                            </div>

                            <!-- Low Price Scenario (Gaming) -->
                            <div
                                class="bg-white p-4 rounded-lg border-2 border-green-400 shadow-md relative overflow-hidden"
                            >
                                <div
                                    class="absolute top-0 right-0 bg-green-400 text-white text-[10px] font-bold px-2 py-1"
                                >
                                    OPTIMIZED
                                </div>
                                <div
                                    class="text-xs font-bold text-green-600 uppercase mb-2"
                                >
                                    Scenario B: "Gaming" the Price
                                </div>
                                <div class="flex justify-between mb-1">
                                    <span class="text-sm text-gray-600"
                                        >Price:</span
                                    >
                                    <span class="font-bold text-gray-800"
                                        >${{ cliffAnalysis.threshold }}</span
                                    >
                                </div>
                                <div class="flex justify-between mb-1">
                                    <span class="text-sm text-gray-600"
                                        >Revenue:</span
                                    >
                                    <span class="font-medium text-gray-500">{{
                                        formatMillions(
                                            cliffAnalysis.threshold * volume
                                        )
                                    }}</span>
                                </div>
                                <div class="flex justify-between mb-1">
                                    <span class="text-sm text-gray-600"
                                        >Royalty ({{
                                            (
                                                cliffAnalysis.lowRate * 100
                                            ).toFixed(0)
                                        }}%):</span
                                    >
                                    <span class="font-bold text-green-600">{{
                                        formatMillions(
                                            cliffAnalysis.threshold *
                                                volume *
                                                cliffAnalysis.lowRate
                                        )
                                    }}</span>
                                </div>
                                <div
                                    class="mt-2 pt-2 border-t border-gray-100 flex justify-between"
                                >
                                    <span
                                        class="text-sm font-bold text-gray-700"
                                        >Net Profit:</span
                                    >
                                    <span class="font-bold text-green-700">{{
                                        formatMillions(
                                            cliffAnalysis.gamingProfit
                                        )
                                    }}</span>
                                </div>
                            </div>
                        </div>

                        <div class="mt-4 text-center">
                            <span
                                class="inline-block bg-red-800 text-white text-sm font-bold px-4 py-2 rounded-full shadow-sm"
                            >
                                Incentive to Undersell:
                                {{
                                    formatMillions(
                                        cliffAnalysis.gamingProfit -
                                            results[0].data.compTotal
                                    )
                                }}
                                Gain
                            </span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- SMOOTHING BANNER -->
            <div
                v-if="useSmoothing"
                class="mb-8 border-t border-green-100 bg-green-50/50 p-6 flex items-center gap-4 rounded-b-xl"
            >
                <div class="bg-green-100 p-2 rounded-full">
                    <!-- Icon: Check/Activity -->
                    <svg
                        xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)"
                        width="24"
                        height="24"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="currentColor"
                        stroke-width="2"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        class="text-green-600"
                    >
                        <polyline points="20 6 9 17 4 12"></polyline>
                    </svg>
                </div>
                <div>
                    <h3 class="text-sm font-bold text-green-800">
                        Smoothing Active
                    </h3>
                    <p class="text-sm text-green-700">
                        Royalty scales linearly. Earning more revenue always
                        equals more profit. No incentive to game.
                    </p>
                </div>
            </div>
        </div>

        <!-- Results Cards -->
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div
                v-for="item in results"
                :key="item.id"
                class="bg-white rounded-xl shadow-md overflow-hidden border border-gray-100 flex flex-col"
            >
                <div
                    class="p-5 border-b border-gray-100 bg-gray-50 flex justify-between items-center"
                >
                    <div>
                        <h3 class="font-bold text-lg text-gray-800">
                            {{ item.title }}
                        </h3>
                        <p class="text-sm text-gray-500">{{ item.desc }}</p>
                    </div>
                    <div class="text-right">
                        <div
                            class="text-xs text-gray-400 uppercase tracking-wider mb-1"
                        >
                            Royalty Rate
                        </div>
                        <div class="text-xl font-black text-blue-600">
                            {{ formatPct(item.data.royaltyRate) }}
                        </div>
                    </div>
                </div>

                <div class="p-6">
                    <!-- Big Numbers -->
                    <div class="grid grid-cols-2 gap-8 mb-6">
                        <div
                            class="text-center p-4 bg-green-50 rounded-lg border border-green-100"
                        >
                            <span
                                class="block text-green-800 text-xs font-bold uppercase tracking-wide mb-1"
                                >Govt Take</span
                            >
                            <span
                                class="block text-2xl md:text-3xl font-black text-green-700 tracking-tight"
                                >{{ formatMillions(item.data.govTotal) }}</span
                            >
                            <span class="text-xs text-green-600/70"
                                >Total Annual</span
                            >
                        </div>
                        <div
                            class="text-center p-4 bg-blue-50 rounded-lg border border-blue-100"
                        >
                            <span
                                class="block text-blue-800 text-xs font-bold uppercase tracking-wide mb-1"
                                >Company Net</span
                            >
                            <span
                                class="block text-2xl md:text-3xl font-black tracking-tight"
                                :class="
                                    item.data.compTotal < 0
                                        ? 'text-red-600'
                                        : 'text-blue-700'
                                "
                            >
                                {{ formatMillions(item.data.compTotal) }}
                            </span>
                            <span class="text-xs text-blue-600/70"
                                >Total Annual</span
                            >
                        </div>
                    </div>

                    <!-- Split Bar -->
                    <div class="mb-2">
                        <div
                            class="flex h-4 w-full rounded-full overflow-hidden bg-gray-200"
                        >
                            <div
                                class="bg-green-500"
                                :style="{ width: item.data.govShare + '%' }"
                            ></div>
                            <div
                                class="bg-blue-600"
                                :style="{ width: item.data.compShare + '%' }"
                            ></div>
                        </div>
                        <div
                            class="flex justify-between text-xs font-bold text-gray-500 mt-1"
                        >
                            <span
                                >Govt {{ item.data.govShare.toFixed(1) }}%</span
                            >
                            <span
                                >Company
                                {{ item.data.compShare.toFixed(1) }}%</span
                            >
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
