<script>
    import * as d3 from 'd3';
    import { onMount } from 'svelte';

    export let index, width, height;
    console.log(index);
    // Updated Data
    const per_career_stats = [
        { percentage_points: 'Points', ham: parseFloat((0.598955647 * 100).toFixed(2)), sch: parseFloat((0.379176755 * 100).toFixed(2)) },
        { percentage_wins: 'Wins', ham: parseFloat((0.309309309 * 100).toFixed(2)), sch: parseFloat((0.297385621 * 100).toFixed(2)) },
        { percentage_fl: 'Fastest Laps', ham: parseFloat((0.195195195 * 100).toFixed(2)), sch: parseFloat((0.251633987 * 100).toFixed(2)) },
        { percentage_pod: 'Podiums', ham: parseFloat((0.591591592 * 100).toFixed(2)), sch: parseFloat((0.506535948 * 100).toFixed(2)) },
        { percentage_poles: 'Poles', ham: parseFloat((0.312312312 * 100).toFixed(2)), sch: parseFloat((0.222222222 * 100).toFixed(2)) }
    ];

    const career_stats = [
        { points: 'Points Count', ham: 4646, sch: 1566 },
        { wins: 'Wins Count', ham: 103, sch: 91 },
        { fl: 'Fastest Laps Count', ham: 65, sch: 77 },
        { pod: 'Podiums Count', ham: 197, sch: 155 },
        { poles: 'Poles Count', ham: 104, sch: 68 }
    ];

    let container;
    let svg;
    let tooltip;
    let bar1;
    let bar2;
    const barHeight = 50;
    let xScale;

    function handleBarMouseOver(event, d, key) {
        // Create a group for the bar and text
        const group = svg.append('g').attr('class', 'bar-group');

        // Highlight the hovered bar
        const currentBar = group.append('rect')
            .attr('x', +event.currentTarget.getAttribute('x'))
            .attr('y', +event.currentTarget.getAttribute('y'))
            .attr('width', +event.currentTarget.getAttribute('width'))
            .attr('height', +event.currentTarget.getAttribute('height'))
            .attr('fill', 'orange');

        // Add text to the group
        group.append('text')
            .attr('class', 'highlighted-text')
            .attr('x', +currentBar.attr('x') + +currentBar.attr('width') / 2)
            .attr('y', +currentBar.attr('y') + barHeight / 2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle')
            .attr('fill', 'black')
            .text(`${d3.format(".2f")(d[key])}%`);
    }

    function handleBarMouseOut(event, d, key) {
        // Delay the removal of the group containing the bar and text
        setTimeout(() => {
            // Remove the group containing the bar and text
            svg.selectAll('.bar-group').remove();
        }, 100); // Adjust the delay time as needed
    }
    onMount(() => {
        console.log('DriverCareer component mounted!');
        //const { index, width, height } = $$props;
        const margin = { top: 20, right: 20, bottom: 20, left: 20 };
        const widthPlsChart = width * 0.95 - margin.left - margin.right;
        const heightPlsChart = height * 0.75 - margin.top - margin.bottom;
        const extraPadding = 50; 

        // Select the container div after the component is mounted
        container = d3.select('#chart-container-1');
        // Remove any existing SVG elements inside the container
        container.selectAll('svg').remove();

        // Append a new SVG with the specified dimensions
        svg = container.append('svg')
            .attr('width', widthPlsChart + margin.left + margin.right)
            .attr('height', heightPlsChart + margin.top + margin.bottom)
            .append('g')
            .attr('transform', `translate(${margin.left},${margin.top})`);

        // Scales
        const maxDataValue = d3.max(per_career_stats, d => Math.max(d.ham, d.sch));
        xScale = d3.scaleLinear()
            .domain([-maxDataValue - extraPadding, maxDataValue + extraPadding])
            .range([0, widthPlsChart]);

        const yScale = d3.scaleBand()
            .domain(per_career_stats.map(d => Object.keys(d)[0]))
            .range([0, heightPlsChart])
            .padding(0.1);

        tooltip = d3.select("body").append("div")
            .attr("class", "tooltip")
            .style("opacity", 0);
        // Calculate the space between bar1 and bar2 for labelAttr
        const spaceBetweenBars = 70;

        // Draw bars
        bar1 = svg.selectAll('.bar1')
            .data(per_career_stats)
            .enter().append('rect')
            .attr('class', 'bar1')
            .attr('x', d => xScale(Math.min(0, d.ham)) + spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#5CA4FF');

        // Add labels for bar1
        svg.selectAll('.label1')
            .data(per_career_stats)
            .enter().append('text')
            .attr('class', 'label1')
            .attr('x', widthPlsChart - 30)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .attr('text-anchor', 'end')
            .each(function (d, i) {
                const label = d3.select(this);
                if (Object.values(per_career_stats[i])[1] >= Object.values(per_career_stats[i])[2]) {
                    label.append('tspan')
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[1]}`)
                        .attr('fill', '#b0d3ff')
                        .append('tspan')
                        .html('<tspan>&#x1F3C6;</tspan>');
                } else {
                    label.append('tspan')
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[1]}`);
                }
            });

            
        // Add labels between bar1 and bar2
        svg.selectAll('.labelAttr')
            .data(per_career_stats)
            .enter().append('text')
            .attr('class', 'labelAttr')
            .attr('x', widthPlsChart/2) // Centered position
            .attr('y', d => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('text-anchor', 'middle') // Center the text
            .attr('fill', "#FAEF5D")
            .text(d => Object.values(d)[0]);

        // Draw bars
        bar2 = svg.selectAll('.bar2')
            .data(per_career_stats)
            .enter().append('rect')
            .attr('class', 'bar2')
            .attr('x', d => xScale(Math.min(0, -d.sch)) - spaceBetweenBars)
            .attr('y', d => yScale(Object.keys(d)[0]))
            .attr('width', 0)
            .attr('height', barHeight)
            .attr('fill', '#8CC84B')
            .on('mouseover', (event, d, i) => handleBarMouseOver(event, d, 'sch'))
            .on('mouseout', (event, d, i) => handleBarMouseOut(event, d, 'sch'));
        

        // Add labels for bar2
        svg.selectAll('.label2')
            .data(per_career_stats)
            .enter().append('text')
            .attr('class', 'label2')
            .attr('x', margin.left + 30)
            .attr('y', (d, i) => yScale(Object.keys(d)[0]) + barHeight / 2)
            .attr('dy', '.35em')
            .attr('fill', "#FAEF5D")
            .attr('cursor', 'pointer')
            .each(function (d, i) {
                const label = d3.select(this);
                if (Object.values(per_career_stats[i])[2] >= Object.values(per_career_stats[i])[1]) {
                    label.append('tspan')
                        .html('<tspan>&#x1F3C6;</tspan>') // Unicode for the trophy symbol
                        .append('tspan')
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[2]}`)
                        .attr('fill', '#a7ed5a');
                } else {
                    label.append('tspan')
                        .text(` ${Object.values(career_stats[i])[0]}: ${Object.values(career_stats[i])[2]}`);
                }
            });
    
        const scaleRight = d3.scaleLinear()
            .domain([maxDataValue + extraPadding, 0])
            .range([-spaceBetweenBars * 2 + margin.left * 2, widthPlsChart/2 - spaceBetweenBars]);

        const xAxis1 = svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', 'translate(0,' + (barHeight + 500) + ')')
            .call(d3.axisTop(scaleRight).tickValues([0, 20, 40, 60, 80])
            .tickFormat(d => (d <= 100) ? (d + '%') : null))
            .selectAll('text')  // Select all tick text elements
            .attr('dx', '0.5em'); 

        // Customize the tick labels appearance
        xAxis1.selectAll('text')
            .style('text-anchor', 'end')
            .attr('dx', '-.8em')
            .attr('dy', '.15em');

        const scaleLeft = d3.scaleLinear()
            .domain([0, maxDataValue + extraPadding])
            .range([widthPlsChart / 2 + spaceBetweenBars, widthPlsChart+ 2*spaceBetweenBars - margin.right * 2]);

        const xAxis2 = svg.append('g')
            .attr('class', 'x-axis')
            .attr('transform', 'translate(0,' + (barHeight + 500) + ')')
            .call(d3.axisTop(scaleLeft).tickValues([0, 20, 40, 60, 80])
            .tickFormat(d => (d <= 100) ? (d + '%') : null))
            .selectAll('text')  // Select all tick text elements
            .attr('dx', '0.5em'); 

        // Customize the tick labels appearance
        xAxis2.selectAll('text')
            .style('text-anchor', 'end')
            .attr('dx', '-.8em')
            .attr('dy', '.15em');
    });
</script>

<div id="chart-container-1"></div>

<style>
    #chart-container-1{
        opacity: 0;
        transition: opacity 2s, visibility 2s;

    }

    #chart-container-1 {
        fill:aquamarine;
    }
</style>