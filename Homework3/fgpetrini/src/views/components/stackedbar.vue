<template>
    <div class="bar_container">
        <h1 id="bar_title"> {{ plot_title }} </h1>
        <div id="stacked-bar"></div>
    </div>
</template>

<script>
    import * as d3 from "d3";
    
    export default {
        name: 'StackedBar',
        data() {
            return {
                satisfaction_data : [],
                loyalty_data : [],
                id : "#stacked-bar",
                loyalty_bins : ["Disloyal Customers", "Loyal Customers"],
                satisfaction_bins : ["Dissatisfied Customers", "Satisfied Customers"],
                color_dict : {"default" : ["#999999", "#ef8a62"],
                                "cb_accessible" : ["#67a9cf", "#ef8a62"] },
                plot_title : "Respondent Satisfaction By Travel Class",
                current_filter : "default",
                color_change : false,
            }
        },
        props:{
            myStackedBarData: Array,
            height : Number,
            width : Number,
            dd_option: String,
            radio_option: String,
        },
        mounted(){
            let pixels = String(Math.round(this.width / 40));
            document.getElementById("bar_title").style.fontSize = pixels+"px";
            this.satisfaction_data = this.getStackedClassBins(this.myStackedBarData, "satisfaction");
            this.drawStackedBar(this.satisfaction_data);
            this.loyalty_data = this.getStackedClassBins(this.myStackedBarData, "Customer Type");
        },
        methods: {
            drawStackedBar(data) {
                if(this.dd_option == "Dis_Vs_Sat") {
                    this.StackedChartNew(this.satisfaction_data, {
                        x: d => d.key,
                        y: d => d.value,
                        z: d => d.category,
                        yLabel: "↑ Number of Respondents",
                        zDomain: this.satisfaction_bins,
                        colors: this.color_dict[this.radio_option],
                        width: this.width,
                        height: this.height
                    });
                } else {
                    this.StackedChartNew(this.loyalty_data, {
                        x: d => d.key,
                        y: d => d.value,
                        z: d => d.category,
                        yLabel: "↑ Number of Respondents",
                        zDomain: this.loyalty_bins,
                        colors: this.color_dict[this.radio_option],
                        width: this.width,
                        height: this.height
                    });
                }
            },
            // Copyright 2021 Observable, Inc.
            // Released under the ISC license.
            // https://observablehq.com/@d3/stacked-bar-chart
            StackedChartNew(data, {
                x = (d, i) => i, // given d in data, returns the (ordinal) x-value
                y = d => d, // given d in data, returns the (quantitative) y-value
                z = () => 1, // given d in data, returns the (categorical) z-value
                title, // given d in data, returns the title text
                marginTop = 50, // top margin, in pixels
                marginRight = 20, // right margin, in pixels
                marginBottom = 30, // bottom margin, in pixels
                marginLeft = 40, // left margin, in pixels
                width = 640, // outer width, in pixels
                height = 400, // outer height, in pixels
                xDomain, // array of x-values
                xRange = [marginLeft, width - marginRight], // [left, right]
                xPadding = 0.1, // amount of x-range to reserve to separate bars
                yType = d3.scaleLinear, // type of y-scale
                yDomain, // [ymin, ymax]
                yRange = [height - marginBottom, marginTop], // [bottom, top]
                zDomain, // array of z-values
                offset = d3.stackOffsetDiverging, // stack offset method
                order = d3.stackOrderNone, // stack order method
                yFormat, // a format specifier string for the y-axis
                yLabel, // a label for the y-axis
                colors = d3.schemeTableau10, // array of colors
                } = {}) {
                // Compute values.
                const X = d3.map(data, x);
                const Y = d3.map(data, y);
                const Z = d3.map(data, z);

                // Compute default x- and z-domains, and unique them.
                if (xDomain === undefined) xDomain = X;
                if (zDomain === undefined) zDomain = Z;
                xDomain = new d3.InternSet(xDomain);
                zDomain = new d3.InternSet(zDomain);

                // Omit any data not present in the x- and z-domains.
                const I = d3.range(X.length).filter(i => xDomain.has(X[i]) && zDomain.has(Z[i]));

                // Compute a nested array of series where each series is [[y1, y2], [y1, y2],
                // [y1, y2], …] representing the y-extent of each stacked rect. In addition,
                // each tuple has an i (index) property so that we can refer back to the
                // original data point (data[i]). This code assumes that there is only one
                // data point for a given unique x- and z-value.
                const series = d3.stack()
                    .keys(zDomain)
                    .value(([x, I], z) => Y[I.get(z)])
                    .order(order)
                    .offset(offset)
                    (d3.rollup(I, ([i]) => i, i => X[i], i => Z[i]))
                    .map(s => s.map(d => Object.assign(d, {i: d.data[1].get(s.key)})));

                // Compute the default y-domain. Note: diverging stacks can be negative.
                if (yDomain === undefined) yDomain = d3.extent(series.flat(2));

                // Construct scales, axes, and formats.
                const xScale = d3.scaleBand(xDomain, xRange).paddingInner(xPadding);
                const yScale = yType(yDomain, yRange);
                const color = d3.scaleOrdinal(zDomain, colors);
                const xAxis = d3.axisBottom(xScale).tickSizeOuter(0);
                const yAxis = d3.axisLeft(yScale).ticks(height / 60, yFormat);

                // Compute titles.
                if (title === undefined) {
                    const formatValue = yScale.tickFormat(100, yFormat);
                    title = i => `${formatValue(Y[i])}` + " " + `${Z[i]}`;
                } else {
                    const O = d3.map(data, d => d);
                    const T = title;
                    title = i => T(O[i], i, data);
                }
                const svg = d3.select(this.id).append("svg")
                    .attr("width", width)
                    .attr("height", height)
                    .attr("viewBox", [0, 0, width, height])
                    .attr("style", "max-width: 100%; height: auto; height: intrinsic;");

                svg.append("g")
                    .attr("transform", `translate(${marginLeft},0)`)
                    .call(yAxis)
                    .call(g => g.select(".domain").remove())
                    .call(g => g.selectAll(".tick line").clone()
                        .attr("x2", width - marginLeft - marginRight)
                        .attr("stroke-opacity", 0.1))
                    .call(g => g.append("text")
                        .attr("x", -marginLeft)
                        .attr("y", 40)
                        .attr("fill", "currentColor")
                        .attr("text-anchor", "start")
                        .text(yLabel));

                let topObject = this;

                const bar = svg.append("g")
                    .selectAll("g")
                    .data(series)
                    .join("g")
                    .attr("fill", ([{i}]) => color(Z[i]))
                    .selectAll("rect")
                    .data(d => d)
                    .join("rect")
                    .attr("x", ({i}) => xScale(X[i]))
                    .attr("y", ([y1, y2]) => Math.min(yScale(y1), yScale(y2)))
                    .attr("height", ([y1, y2]) => Math.abs(yScale(y1) - yScale(y2)))
                    .attr("width", xScale.bandwidth())
                    .attr("bar-category", (d) => {
                        if(d['i'] == 0 || d['i'] == 1) {
                            return "Eco";
                        } else if(d['i'] == 2 || d['i'] == 3) {
                            return "Eco Plus";
                        } else {
                            return "Business";
                        }
                    })
                    .on("mouseover", function(d,i) {
                        let target_param = i['data'][0];
                            d3.selectAll("rect")
                                .filter(function() {
                                    return d3.select(this).attr("bar-category") != target_param
                                })
                            .style("transition", "opacity 0.8s")
                            .attr('opacity', 0.5);
                        })
                    .on("mouseout", function(d) {
                        d3.selectAll("rect")
                            .style("transition", "opacity 0.8s")
                            .attr('opacity', 1.0);
                    })
                    .on("click", function(d,i) {
                        console.log(current_param);
                        let target_param = i['data'][0];
                        let all_rect = d3.selectAll("rect");
                        let target_bar = all_rect.filter(function() {
                            return d3.select(this).attr("bar-category") == target_param;
                        });
                        let already_selected = target_bar.filter(function() {
                            return (d3.select(this).attr("stroke-width") == "2");
                        })["_groups"][0].length > 0;
                        // Clear any previous modifications
                        all_rect.attr("stroke", "black")
                                .attr("stroke-width", 0)
                        var current_param = "default";
                        console.log(already_selected);
                        if(!already_selected) {
                            target_bar.attr("stroke", "black")
                                .attr("stroke-width", 2);
                            current_param = target_param
                        }
                        console.log(topObject);
                        topObject.current_filter = current_param;
                        topObject.$parent.stackedBarChanged(current_param);
                    });

                // Redraw outline when color filter is changed
                if(this.color_change && this.current_filter != "default") {
                    let target_bar = d3.selectAll("rect");
                    let filter = this.current_filter;
                    target_bar.each(function(p,j){
                        console.log(p["data"][0]);
                        if(p["data"][0] == filter)
                            d3.select(this).attr("stroke", "black").attr("stroke-width", 2);
                    });
                } else {
                    // A non color change has taken place
                    this.current_filter = "default";
                }

                if (title) bar.append("title")
                    .text(({i}) => title(i));

                svg.append("g")
                    .attr("transform", `translate(0,${yScale(0)})`)
                    .call(xAxis);
            },
            getStackedClassBins(input, index_category) {
                let retList = [];
                let dict = {}
                if(index_category == "Customer Type") {
                    dict = {"Eco" : {"Loyal Customer": 0,"disloyal Customer": 0},
                            "Eco Plus" : {"Loyal Customer": 0, "disloyal Customer": 0},
                            "Business" : {"Loyal Customer": 0, "disloyal Customer": 0},
                    };

                    for(let i = 0; i < input.length; i++) {
                        dict[input[i]["Class"]][input[i][index_category]] += 1;
                    }

                    for(let key in dict) {
                        let value = dict[key]["disloyal Customer"];
                        let category = "Disloyal Customers";
                        retList.push({key, category, value});
                        
                        value = dict[key]["Loyal Customer"];
                        category = "Loyal Customers";
                        retList.push({key, category, value});
                    }
                } else {
                    dict = {"Eco" : {"satisfied": 0, "neutral or dissatisfied": 0},
                            "Eco Plus" : {"satisfied": 0, "neutral or dissatisfied": 0},
                            "Business" : {"satisfied": 0, "neutral or dissatisfied": 0},
                    };

                    for(let i = 0; i < input.length; i++) {
                        dict[input[i]["Class"]][input[i][index_category]] += 1;
                    }

                    for(let key in dict) {
                        let value = dict[key]["neutral or dissatisfied"];
                        let category = "Dissatisfied Customers";
                        retList.push({key, category, value});
                        
                        value = dict[key]["satisfied"];
                        category = "Satisfied Customers";
                        retList.push({key, category, value});
                    }
                }
                return retList;
            },
            updatePlot(color_change) {
                d3.selectAll("#stacked-bar svg").remove();
                this.color_change = color_change;
                if(this.dd_option == "Dis_Vs_Sat") {
                    this.plot_title = "Respondent Satisfaction By Travel Class";
                    this.drawStackedBar(this.satisfaction_data);
                }
                else {
                    this.plot_title = "Respondent Loyalty By Travel Class";
                    this.drawStackedBar(this.loyalty_data);
                }
            },
        }
    }

</script>


<style scoped>
.bar_container {
    width: 100%;
    height: 100%;
}
#stacked-bar {
    position: relative;
    top: -41px;
    left: 0;
    z-index: 90;
}

#bar_title {
    position: relative;
    top: 0;
    left: 0;
    padding-left: 50px;
    z-index: 100;
    font-size: 20px;
    font-weight: bold;
    text-decoration: underline;
    text-align: center;
}
</style>