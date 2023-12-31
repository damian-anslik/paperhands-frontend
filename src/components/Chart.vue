<template>
    <apexchart type="candlestick" :options="chartOptions" :series="series" height="100%" width="100%" ref="chart" />
</template>

<script>
import controller from '@/controller'
export default {
    name: "Apex charts",
    computed: {
        activeSymbol() {
            return this.$store.state.activeSymbol
        }
    },
    mounted() {
        if (this.activeSymbol === '') return
        this.requestHistoricalData(this.activeSymbol)
    },
    watch: {
        activeSymbol: function (symbol) {
            if (symbol === '') return
            this.chartIsLoading(symbol)
            this.requestHistoricalData(symbol)
        }
    },
    methods: {
        requestHistoricalData(symbol, name) {
            controller.getHistoricalData(symbol)
                .then(response => {
                    const bars = response.data.bars.map(bar => {
                        return {
                            x: new Date(bar.t),
                            y: [bar.o, bar.h, bar.l, bar.c]
                        }
                    })
                    this.updateSeries([
                        {
                            data: bars
                        }
                    ])
                    this.updateChartOptions({
                        title: {
                            text: `${symbol}`
                        },
                        noData: {
                            text: `No data available for ${symbol}`
                        }
                    })
                })
                .catch(error => {
                    console.log(error);
                })
        },
        chartIsLoading(symbol) {
            this.updateSeries([])
            this.updateChartOptions({
                title: {
                    text: ''
                },
                noData: {
                    text: 'Loading...'
                }
            })
        },
        updateChartOptions(newOptions) {
            this.$refs.chart.updateOptions(newOptions)
        },
        updateSeries(newSeries) {
            this.$refs.chart.updateSeries(newSeries, true)
        }
    },
    data: () => ({
        chartOptions: {
            chart: {
                type: 'candlestick',
                toolbar: {
                    show: true,
                    offsetX: -4,
                    tools: {
                        download: false,
                        selection: true,
                        zoom: true,
                        zoomin: true,
                        zoomout: true,
                        pan: true,
                        reset: true
                    },
                },
                animations: {
                    enabled: false
                }
            },
            noData: {
                text: 'Select a symbol to view chart',
            },
            grid: {
                borderColor: '#333',
            },
            title: {
                align: 'left'
            },
            xaxis: {
                type: 'datetime',
            },
            yaxis: {
                show: true,
                tooltip: {
                    enabled: true,
                    style: {
                        fontSize: '12px',
                        fontFamily: undefined
                    },
                },
                labels: {
                    formatter: function (val) {
                        return Math.floor(val)
                    }
                }
            },
            tooltip: {
                theme: 'dark'
            }
        },
        series: [],
    }),
};
</script>