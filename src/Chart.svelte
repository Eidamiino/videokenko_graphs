<script>
    import { onMount } from "svelte";
    import { Chart } from "chart.js/auto";

    let chartBind;
    let chart;
    let chartData;

    const Utils = {
        getRandomMaxTime() {
            return Math.floor(Math.random() * 1000) + 2600;
        },
        generateTimeSegments(maxTime, segments) {
            const labels = [];
            for (let i = 0; i < segments; i++) {
                const lowerBound = Math.floor(i * (maxTime / segments));
                labels.push(lowerBound);
            }
            return labels;
        },
        generateViewsForSegments(segments) {
            const viewsInSegment = new Array(segments).fill(0);
            for (let i = 0; i < segments; i++) {
                viewsInSegment[i] = Math.floor(Math.random() * 501 + 500);
            }
            return viewsInSegment;
        },
        CHART_COLORS: {
            blue: "rgb(54, 162, 235)",
        },
        transparentize(color, opacity = 0.5) {
            return color.replace("rgb", "rgba").replace(")", `, ${opacity})`);
        },
    };

    function formatTime(seconds) {
        const minutes = Math.floor(seconds / 60);
        const remainingSeconds = seconds % 60;
        return `${minutes}:${remainingSeconds < 10 ? "0" : ""}${remainingSeconds}`;
    }

    function generateChartData() {
        const videoLength = Utils.getRandomMaxTime();
        const segments = Math.floor(videoLength / 5);
        const randomViews = Utils.generateViewsForSegments(segments);
        //dummy input data, intervals of 5

        const viewsBySecond = randomViews.flatMap((num) =>
            Array(5).fill(num / 5),
        );

        const newSegmentLength = 30;

        const newViews = [];
        for (let i = 0; i < viewsBySecond.length; i += newSegmentLength) {
            const segment = viewsBySecond.slice(i, i + newSegmentLength);
            const segmentSum = segment.reduce((x, val) => x + val, 0);
            newViews.push(segmentSum);
        }

        const timeLabels = Utils.generateTimeSegments(videoLength, (videoLength/newSegmentLength));

        return {
            labels: timeLabels,
            datasets: [
                {
                    label: "Views",
                    data: newViews,
                    backgroundColor: Utils.transparentize(
                        Utils.CHART_COLORS.blue,
                        0.7,
                    ),
                    borderColor: Utils.CHART_COLORS.blue,
                    borderWidth: 1,
                },
            ],
        };
    }

    chartData = generateChartData();

    onMount(() => {
        const ctx = chartBind.getContext("2d");
        chart = new Chart(ctx, {
            type: "bar",
            data: chartData,
            options: {
                scales: {
                    x: {
                        title: {
                            display: true,
                            text: "Time",
                        },
                        ticks: {
                            autoSkip: true,
                            maxTicksLimit: 10,
                            callback: function (value) {
                                if (typeof value === "number") {
                                    return formatTime(
                                        this.getLabelForValue(value),
                                    );
                                }
                                return value;
                            },
                        },
                    },
                    y: {
                        beginAtZero: false,
                        title: {
                            display: true,
                            text: "Views",
                        },
                        ticks: {
                            stepSize: 100,
                        },
                    },
                },
            },
        });
    });

    function updateChart() {
        chart.data = generateChartData();
        chart.update();
    }
</script>

<button on:click={updateChart}>New data</button>

<canvas bind:this={chartBind} width="1200" height="800"></canvas>

<style>
    canvas {
        max-width: 100%;
        height: auto;
    }

    button {
        margin-bottom: 20px;
        padding: 10px 20px;
        font-size: 16px;
        background-color: #007bff;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    }

    button:hover {
        background-color: #0056b3;
    }
</style>
