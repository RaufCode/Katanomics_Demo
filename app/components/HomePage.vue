<template>
    <div class="bg-white text-gray-800">
        <div class="max-w-4xl mx-auto px-6 py-3 leading-relaxed">
            <!-- Title -->
            <header class="mb-3">
                <h1
                    class="text-4xl md:text-5xl font-extrabold tracking-tight text-gray-900 mb-3"
                >
                    De-Katanomising Africa’s Mineral Fiscal Policy Through DPI
                    Tech
                </h1>
                <p class="text-lg text-gray-900 font-semibold">
                    From static royalties to sovereign intelligence in the age
                    of critical minerals.
                </p>
            </header>

            <!-- INTRO -->
            <section class="prose prose-lg max-w-none mb-6">
                <h2 class="font-bold text-gray-900 mb-4">
                    Introduction: The Myth of Abundance and the Reality of Data
                </h2>

                <p class="mb-4">
                    As concerns over global warming force a shift to battery
                    technologies, a new class of
                    <strong>critical minerals</strong> such as lithium, cobalt,
                    and rare earths has taken center stage. For West Africa,
                    historically tied to older minerals like gold and bauxite,
                    this transition is a make-or-break moment.
                </p>

                <p class="mb-4">
                    Yet, the region is locked into a pernicious
                    <strong>“myth of 30%”</strong>, which suggests that the
                    continent holds a third of the world's resources and needs
                    only to wait for global demand to knock. The truth is that
                    much of the continent is seriously starved of the essential
                    industrial minerals needed for structural economic
                    transformation.
                </p>

                <p class="mb-4">
                    This abundance mindset obscures a precarious reality: the
                    continent is <em>data-poor</em> in an era where geological
                    intelligence is the primary currency of exploration. African
                    governments often sell exploration licenses like lottery
                    tickets without knowing the odds of winning, while foreign
                    entities use AI and mirror geology to extract value from
                    African data to de-risk investments elsewhere.
                </p>

                <p class="mb-4">
                    Furthermore, the fiscal instruments used to capture value,
                    static royalties and manual tax assessments, suffer from
                    <strong>Katanomics</strong>, a structural fracture between
                    political ambition, policy design, and execution. Bridging
                    this gap requires
                    <strong>Sovereign Intelligence</strong> and the deployment
                    of <strong>Digital Public Infrastructure (DPI)</strong>.
                </p>
            </section>

            <!-- CASE STUDY -->
            <section class="prose prose-lg max-w-none mb-6">
                <h2 class="font-bold mb-4">
                    The Ewoyaa Case: Katanomics in Action
                </h2>

                <p class="mb-4">
                    The Ewoyaa Lithium Project in Ghana serves as a prime case
                    study. Unlike the established gold sector, lithium is a
                    greenfield regulatory space. The political narrative of
                    historic value capture collided with the operational reality
                    of the developer, Atlantic Lithium, a junior explorer
                    navigating the Valley of Death between discovery and
                    production.
                </p>

                <p class="mb-4">
                    The state’s initial negotiation for a fixed 10% royalty was
                    jettisoned without any credible account, despite vibrant
                    political debate, a classic symptom of the katanomic
                    syndrome in which political voice does not translate into
                    policy accountability.
                </p>

                <p class="mb-4">
                    The subsequent renegotiated 5% base royalty rate failed to
                    account for extreme price volatility (lithium prices swung
                    from $500 to $6,000 per ton in two years) or the metal’s
                    complex chemistry-economics. Unsurprisingly, it was greeted
                    with civil uproar.
                </p>

                <p class="mb-4">
                    Eventually, a sliding-scale mechanism was introduced. While
                    pragmatic, this shift was presented without serious fiscal
                    modelling, betraying the need for rigorous, data-backed
                    sensitivity analysis. What Ghana received instead was a fiat
                    outcome, a situation begging for
                    <strong>de-katanomisation</strong>.
                </p>
            </section>

            <!-- INTERACTIVE ANALYSIS -->
            <section class="mb-6">
                <h2 class="text-lg font-bold text-gray-900 mb-3">
                    Interactive Fiscal Analysis: Deconstructing the Government
                    Take
                </h2>

                <p class="text-lg text-gray-700 mb-4 max-w-3xl">
                    De-katanomisation means understanding the true impact and
                    import of fiscal regime design and moving beyond headline
                    numbers to model the Effective Government Take under various
                    scenarios. The following interactive modules allow us to
                    toggle the variables that define the state’s revenue versus
                    the operator’s survival.
                </p>

                <p class="text-lg text-gray-700 mb-8 max-w-3xl">
                    Scenario A vs. B: The royalty baseline. First, we analyse
                    the impact of the royalty rate itself. In a market downturn,
                    a high fixed royalty serves as a fiscal bulwark for the
                    state, but the private operator must sacrifice margin.
                    Conversely, a lower baseline offers relief to the investor
                    but reduces the state’s guaranteed off-the-top revenue.
                </p>

                <!-- WEDGE 1 -->
                <div class="mb-5">
                    <h3 class="text-lg font-bold mb-3">
                        Scenario A: Base Royalty (5%), Fixed AISC Costs
                    </h3>
                    <div
                        class="border-2 border-dashed border-gray-300 rounded-xl p-6 text-center text-gray-500"
                    >
                        <FiscalWedge
                            :baseRoyalty="0.05"
                            :fixedAisc="610"
                            :price="1500"
                            :variableAisc="645.7"
                            :volume="500000"
                            :showSmoothingToggle="true"
                        />
                    </div>
                </div>

                <!-- WEDGE 2 -->
                <div class="mb-5">
                    <h3 class="text-lgs font-bold mb-3">
                        Scenario B: Higher Base Royalty (10%), Fixed AISC Costs
                    </h3>
                    <div
                        class="border-2 border-dashed border-gray-300 rounded-xl p-6 text-center text-gray-500"
                    >
                        <FiscalWedge
                            :baseRoyalty="0.1"
                            :fixedAisc="610"
                            :price="3000"
                            :variableAisc="645.7"
                            :volume="500000"
                            :useSmoothing="false"
                        />
                    </div>
                </div>

                <!-- COST BASE -->
                <div class="prose prose-lg max-w-none mb-5">
                    <p class="mb-4">
                        Cost Base: All-In Sustaining Costs (AISC). However,
                        royalty rates do not exist in a vacuum. The All-In
                        Sustaining Cost determines project profitability. In an
                        inflationary environment or during complex geological
                        extraction, costs can spiral, making even a modest
                        royalty rate punitive. These scenarios demonstrate how
                        cost variability interacts with the royalty burden.
                    </p>
                </div>

                <!-- WEDGE 3 -->
                <div class="mb-5">
                    <h3 class="text-lg font-bold mb-3">
                        Scenario C: Base Royalty (5%), Variable AISC Costs
                    </h3>
                    <div
                        class="border-2 border-dashed border-gray-300 rounded-xl p-6 text-center text-gray-500"
                    >
                        <FiscalWedge
                            :baseRoyalty="0.05"
                            aiscMode="variable"
                            :volume="500000"
                            :price="1500"
                        />
                    </div>
                </div>

                <!-- WEDGE 4 -->
                <div class="mb-5">
                    <h3 class="text-lg font-bold mb-3">
                        Scenario D: Higher Base Royalty (10%), Variable AISC
                        Costs
                    </h3>
                    <div
                        class="border-2 border-dashed border-gray-300 rounded-xl p-6 text-center text-gray-500"
                    >
                        <FiscalWedge
                            :baseRoyalty="0.1"
                            aiscMode="variable"
                            :price="3000"
                            :volume="500000"
                        />
                    </div>
                </div>

                <!-- CLIFF EFFECT -->
                <div class="prose prose-lg max-w-none mb-5">
                    <p class="mb-4">
                        Solving the "Cliff Effect" with smoothing. A critical
                        flaw in the bracket or step approach to royalty design
                        adopted by Ghana’s Lands Ministry is the cliff effect.
                        When a royalty rate jumps instantly at a specific price
                        threshold, for example from 7% to 10% at $2,500 per ton,
                        it creates a perverse incentive.
                    </p>

                    <p class="mb-4">
                        To eliminate this, the remedy is a smoothed sliding
                        scale. Instead of steps, the royalty rate is calculated
                        using a continuous interpolation formula. This ensures
                        that earning more revenue always results in higher net
                        profit, removing incentives for transfer pricing
                        manipulation.
                    </p>
                </div>

                <!-- WEDGE 5 -->
                <div>
                    <h3 class="text-lg font-bold mb-3">
                        Scenario E: Smoothed Royalty, Fixed AISC Costs
                    </h3>
                    <div
                        class="border-2 border-dashed border-green-300 bg-green-50 rounded-xl p-6 text-center text-green-700"
                    >
                        <FiscalWedge
                            title="Scenario E: Smoothed Royalty, Fixed AISC Costs"
                            :baseRoyalty="0.05"
                            aiscMode="fixed"
                            :useSmoothing="true"
                            :volume="500000"
                            :price="3050"
                            :royaltyBands="[
                                { price: 1500, rate: 0.05 },
                                { price: 2500, rate: 0.07 },
                                { price: 3000, rate: 0.1 },
                                { price: 4000, rate: 0.12 },
                            ]"
                        />
                    </div>
                </div>
            </section>

            <!-- DIGITAL GAZETTE -->
            <section class="prose prose-lg max-w-none mb-6">
                <h2 class="font-bold mb-4">
                    Operationalizing Policy: The Digital Gazette
                </h2>

                <p class="mb-4">
                    But even the best mathematical model is useless if the
                    administrative machinery cannot implement it. In a katanomic
                    state, complex rules are hard to enforce manually due to
                    weak analytic vigilance. We propose the Digital Gazette, a
                    transformation of mining codes from static PDF documents
                    into computable regulatory logic.
                </p>

                <p class="mb-4">
                    In this architecture, the Minerals Commission publishes
                    fiscal logic in standardized, version-controlled code. This
                    creates an immutable audit trail and allows critical policy
                    audiences, including CSOs, journalists, and analysts, to run
                    simulations and audit regime performance in real time. The
                    state shifts from a passive regulator to an active,
                    automated governor while simplifying compliance for all
                    actors.
                </p>
            </section>

            <!-- DATA SOVEREIGNTY -->
            <section class="prose prose-lg max-w-none mb-6">
                <h2 class="font-bold mb-4">
                    Data Sovereignty and the Enterprise Marketplace
                </h2>

                <p class="mb-4">
                    A crucial component of this infrastructure is the National
                    Data Cloud. By mandating that exploration data be uploaded
                    in machine-readable formats with QA and QC metadata, the
                    state asserts ownership over its geological IP.
                </p>

                <p class="mb-4">
                    These technologies also enable an enterprise mineral data
                    marketplace. In this ecosystem, private and B2B royalties on
                    e-cadastral data used for successful exploration can be
                    tracked and managed effectively. A chain-of-benefits model
                    ensures original data providers are compensated when their
                    data leads to discovery, incentivising investment in
                    high-resolution mapping and AI enrichment.
                </p>
            </section>

            <!-- FELDSPAR -->
            <section class="prose prose-lg max-w-none mb-6">
                <h2 class="font-bold mb-4">
                    The Feldspar Pilot: A Sandbox for Reform
                </h2>

                <p class="mb-4">
                    Harmonising these advanced systems across West Africa is a
                    heavy political lift. To build policy stamina, we propose a
                    low-stakes sandbox using feldspar.
                </p>

                <p class="mb-4">
                    Feldspar is a byproduct of lithium mining and is vital for
                    the region’s ceramic tile industry. Because it is not a
                    security mineral with high geopolitical stakes, it offers an
                    emotional sandbox environment. Governments can test the
                    Digital Gazette, smoothed royalty protocols, and
                    cross-border data rails on feldspar trade between Ghana and
                    Nigeria without threatening national budgets.
                </p>
            </section>

            <!-- CONCLUSION -->
            <section class="prose prose-lg max-w-none">
                <h2 class="font-bold mb-4">
                    Conclusion: From Resource Nationalism to Smart Sovereignty
                </h2>

                <p class="mb-4">
                    The transition from the old deal to the new deal at Ewoyaa
                    is a microcosm of Africa’s broader struggle. While recent
                    regulations show pragmatism, they remain vulnerable to the
                    structural flaws of analogue governance.
                </p>

                <p class="mb-4">
                    By adopting DPI modules such as smoothing algorithms,
                    Digital Gazettes, and sovereign data marketplaces, West
                    African states can transcend resource nationalism and
                    achieve smart sovereignty.
                </p>

                <p class="mb-4">
                    The strategic game is not won by shouting louder at the
                    negotiating table, but by having the digital intelligence to
                    know the odds, own the data, and automate value capture.
                    Starting with the humble feldspar tile, the region can build
                    the institutional muscle required to govern the electric
                    vehicle battery value chain of the future.
                </p>

                <p class="font-semibold text-blue-700 underline cursor-pointer">
                    Read the full whitepaper here:
                </p>
            </section>
        </div>
    </div>
</template>
