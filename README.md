
<html>
<style>
    body {
        font: 11px sans-serif;
    }

    .axis path,
    .axis line {
        fill: none;
        stroke: #000;
        shape-rendering: crispEdges;
    }

    .dot {
        stroke: #000;
    }

    .tooltip {
        position: absolute;
        width: 100;
        height: 28px;
        pointer-events: none;
    }

    h1 {
        text-align: center;
    }

    h2 {
        text-align: left;
    }
    .about {
        font: 12px sans-serif;
        padding: 0.7rem;
    }
    .title, .subtitle, .about {
        color: #989898
    }
    label {
        font: 14px sans-serif;
    }

</style>
    <h1 class="title">Exploring Temperatures By Months</h1>
    <h2 class="subtitle">About This Graph:</h2>
    <p class="about">This graph displays different temperatures across 12 months in 2014 and 2015 in Houston, New York, and Seattle. Each variable is an average of the daily temperatures for the given value.
    For example, Average Record Max Temp is calculated by averaging the record maximum temps for each day in a given month.
    <b>To use this graph</b>, select the desired Y variable from the menu above and click Explore! Hover over each point to view details.</p>
    <!-- create options for dropdown menu -->
    <p><span><label for="y-axis">Select Y Variable: </label></span>
        <select id="yValue">
            <option value="avg_actual_mean_temp">Avg Actual Mean Temp</option>
            <option value="avg_actual_max_temp">Avg Actual Max Temp</option>
            <option value="avg_actual_min_temp">Avg Actual Min Temp</option>
            <option value="avg_record_max_temp">Avg Record Max Temp</option>
            <option value="avg_record_min_temp">Avg Record Min Temp</option>
        </select>
    <button onclick="plotGraph()">Explore!</button>

<body>

        <script src="https://d3js.org/d3.v3.min.js"></script>
        <script src="https://code.jquery.com/jquery-2.1.3.min.js"></script>

        <script>

            function graph(xText, yText) {
                // svg dimensions/set up
                $('svg').remove();
                var margin = { top: 20, right: 100, bottom: 30, left: 40 },
                    width = 900 - margin.left - margin.right,
                    height = 500 - margin.top - margin.bottom;

                // data prep for x values
                var xValue = function (d) { return d[xText]; }, // access data value
                    xScale = d3.scale.linear().range([0, width]), // translate to visual encoding
                    xMap = function (d) { return xScale(d.Month); }, // translade data to display
                    xAxis = d3.svg.axis().scale(xScale).orient("bottom"); // create axis

                // data prep for y values
                var yValue = function (d) { return d[yText]; },
                    yScale = d3.scale.linear().range([height, 0]),
                    yMap = function (d) { return yScale(yValue(d)); },
                    yAxis = d3.svg.axis().scale(yScale).orient("left");

                // function for circle colors
                var assignColor = function (d) { return d.City; },
                    color = d3.scale.category20();
                console.log(color)

                // add the graph canvas to the body of the webpage
                var svg = d3.select("body").append("svg")
                    .attr("width", width + margin.left + margin.right)
                    .attr("height", height + margin.top + margin.bottom)
                    .append("g")
                    .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

                // add the tooltip area to the webpage
                var tooltip = d3.select("body").append("div")
                    .attr("class", "tooltip")
                    .style("opacity", 0)
                    .attr("class", "tooltip")
                    .style("background-color", "#f0f0f0")
                    .style("border", "solid")
                    .style("border-width", "1px")
                    .style("border-radius", "8px")
                    .style("padding", "15px")

                // import data
                // this file calculates the average for each column by month using a pivot (each row is a month rather than a day)
                d3.csv("monthly_avg.csv", function (error, data) {

                    // reformat data
                    data.forEach(function (d) {
                        d[yText] = +d[yText];
                        d[xText] = +d[xText];
                    });

                    // set axis domains
                    xScale.domain([0, 12]);
                    yScale.domain([0, 105]);

                    // append x axis to svg
                    svg.append("g")
                        .attr("class", "x axis")
                        .attr("transform", "translate(0," + height + ")")
                        .call(xAxis)
                        .append("text")
                        .attr("class", "label")
                        .attr("x", width)
                        .attr("y", -6)
                        .style("text-anchor", "end")
                        .text(xText);

                    // append y axis to svg
                    svg.append("g")
                        .attr("class", "y axis")
                        .call(yAxis)
                        .append("text")
                        .attr("class", "label")
                        .attr("transform", "rotate(-90)")
                        .attr("y", 6)
                        .attr("dy", ".71em")
                        .style("text-anchor", "end")
                        .text(yText);

                    // append points and mousing behavior
                    svg.selectAll(".dot")
                        .data(data)
                        .enter().append("circle")
                        .attr("class", "dot")
                        .attr("r", 8)
                        .attr("cx", xMap)
                        .attr("cy", yMap)
                        .style("fill", function (d) { return color(assignColor(d)); })
                        .style("opacity", 0.7)
                            .on("mouseover", function (d) {
                                tooltip.transition()
                                    .duration(600)
                                    .style("opacity", .9);
                                tooltip.html(d["City"] + "<br/>" + "Month: " + d["Month"] + "<br/>"
                                    + "Exact Y Value: " + yValue(d))
                                    .style("left", (d3.event.pageX + 10) + "px")
                                    .style("top", (d3.event.pageY - 28) + "px");
                            })
                            .on("mouseout", function (d) {
                                tooltip.transition()
                                    .duration(800)
                                    .style("opacity", 0);
                            });

                    // draw legend
                    var legend = svg.selectAll(".legend")
                        .data(color.domain())
                        .enter().append("g")
                        .attr("class", "legend")
                        .attr("transform", function (d, i) { return "translate(10," + (i + 10) * 20 + ")"; });

                    // draw legend colored rectangles
                    legend.append("rect")
                        .attr("x", width + 30)
                        .attr("width", 18)
                        .attr("height", 18)
                        .style("fill", color);

                    // draw legend text
                    legend.append("text")
                        .attr("x", width + 80)
                        .attr("y", 9)
                        .attr("dy", ".35em")
                        .style("text-anchor", "end")
                        .text(function (d) { return d; })
                });
            }

            graph('Month', 'avg_actual_mean_temp');

            function plotGraph() {
                graph($('#xValue').val(), $('#yValue').val());
            }
        </script>
</body>
<html>
