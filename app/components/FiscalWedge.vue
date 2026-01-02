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
            default: "fixed",
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
   LOCAL STATE
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
   EFFECTIVE AISC
======================= */
    const effectiveAisc = computed(() => {
        if (props.aiscMode === "fixed") return props.fixedAisc;

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
   RESULTS
======================= */
    const results = computed(() => {
        const res = calculateSplit(price.value, effectiveAisc.value);

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
   CLIFF EFFECT
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
    <div class="max-w-5xl mx-auto rounded-xl font-sans text-gray-800">
        <!-- Header -->
        <div class="mb-6 text-center">
            <h2 class="font-bold text-gray-900 mb-4">
                Lithium Royalty: The "Cliff Effect"
            </h2>

            <p class="text-gray-600 max-w-2xl mx-auto mb-4">
                Visualize how fixed royalty thresholds create perverse
                incentives to "game" the price. Compare the
                <strong>Step (Cliff)</strong> model vs. a
                <strong>Smoothed</strong> model.
            </p>
        </div>

        <!-- MAIN PANEL -->
        <section
            class="bg-white rounded-xl shadow-lg border border-gray-200 mb-6"
        >
            <div class="p-6 md:p-8">
                <div class="flex flex-col md:flex-row gap-8 mb-6">
                    <!-- Volume -->
                    <div class="flex-1">
                        <label
                            class="block text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide"
                        >
                            Annual Production (Tonnes)
                        </label>

                        <input
                            type="number"
                            v-model.number="volume"
                            class="block w-full text-lg border-gray-300 rounded-lg px-4 py-3 bg-gray-50 border focus:ring-blue-500 focus:border-blue-500"
                        />

                        <p class="text-xs text-gray-400 mt-2 mb-4">
                            Adjust to see impact on annual revenues (e.g.,
                            500,000t).
                        </p>
                    </div>

                    <!-- Toggle -->
                    <div class="flex-1">
                        <label
                            class="block text-sm font-bold text-gray-700 mb-2 uppercase tracking-wide"
                        >
                            Royalty Regime
                        </label>

                        <div class="flex bg-gray-100 p-1 rounded-lg mb-4">
                            <button
                                @click="useSmoothing = false"
                                class="flex-1 py-3 px-4 rounded-md text-sm font-bold transition"
                                :class="
                                    !useSmoothing
                                        ? 'bg-white text-red-600 shadow'
                                        : 'text-gray-500'
                                "
                            >
                                Step (Cliff)
                            </button>

                            <button
                                @click="useSmoothing = true"
                                class="flex-1 py-3 px-4 rounded-md text-sm font-bold transition"
                                :class="
                                    useSmoothing
                                        ? 'bg-white text-green-600 shadow'
                                        : 'text-gray-500'
                                "
                            >
                                Smoothed
                            </button>
                        </div>

                        <p
                            v-if="!useSmoothing"
                            class="text-xs text-gray-400 text-center mb-4"
                        >
                            Current Ghana model (fixed bands)
                        </p>

                        <p
                            v-else
                            class="text-xs text-gray-400 text-center mb-4"
                        >
                            Proposed linear interpolation
                        </p>
                    </div>
                </div>

                <!-- Price Slider -->
                <div class="mb-6">
                    <div class="flex justify-between items-end mb-4">
                        <label
                            class="text-sm font-bold text-gray-700 uppercase tracking-wide"
                        >
                            Market Price (USD/t)
                        </label>

                        <div class="text-3xl font-black text-blue-600">
                            ${{ formatNum(price) }}
                        </div>
                    </div>

                    <input
                        type="range"
                        min="1000"
                        max="4000"
                        step="10"
                        v-model.number="price"
                        class="w-full h-4 bg-gray-200 rounded-full appearance-none cursor-pointer accent-blue-600"
                    />

                    <div
                        class="flex justify-between text-xs font-medium text-gray-400 mt-2"
                    >
                        <span>$1,000</span>
                        <span>$2,500</span>
                        <span>$3,000</span>
                        <span>$4,000</span>
                    </div>
                </div>
            </div>
        </section>

        <!-- RESULTS -->
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
                        <!-- Govt -->
                        <div
                            class="text-center p-4 bg-green-50 rounded-lg border border-green-100"
                        >
                            <span
                                class="block text-xs font-bold uppercase tracking-wide text-gray-700 mb-1"
                            >
                                Govt Take
                            </span>
                            <span
                                class="block text-lg md:text-3xl font-black text-green-700 tracking-tight"
                            >
                                {{ formatMillions(item.data.govTotal) }}
                            </span>
                            <span class="text-xs text-green-600/70">
                                Total Annual
                            </span>
                        </div>

                        <!-- Company -->
                        <div
                            class="text-center p-4 bg-blue-50 rounded-lg border border-blue-100"
                        >
                            <span
                                class="block text-xs font-bold uppercase tracking-wide text-gray-700 mb-1"
                            >
                                Company Net
                            </span>
                            <span
                                class="block text-lg md:text-3xl font-black tracking-tight"
                                :class="
                                    item.data.compTotal < 0
                                        ? 'text-red-600'
                                        : 'text-blue-700'
                                "
                            >
                                {{ formatMillions(item.data.compTotal) }}
                            </span>
                            <span class="text-xs text-blue-600/70">
                                Total Annual
                            </span>
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
                            <span>
                                Govt {{ item.data.govShare.toFixed(1) }}%
                            </span>
                            <span>
                                Company {{ item.data.compShare.toFixed(1) }}%
                            </span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>
