---
style: cfr-style.css
---

```js
import * as d3 from "npm:d3";
```

<!-- Project Description  -->
<section class="project-description">
  <h1>Guac Shock: Avocado Price Scrollytelling Visualization</h1>
  <p>
  During my internship as a Data Visualization Intern at the <a href="https://www.cfr.org/">Council on Foreign Relations</a>, I transformed the static <a href="https://www.cfr.org/article/guac-shock-understanding-tariffs-avocados">"Guac Shock: Understanding Tariffs on Avocados"</a> graphic into a dynamic, D3-powered scrollytelling visualization in Observable. As users scroll, an animated stacked-bar chart breaks down import cost, tariff, and profit on avocados from Mexico—making the component fully reusable across datasets. Special thanks to <a href="https://www.cfr.org/staff#:~:text=UI/UX%20Designer-,Michael%20Bricknell,-Data%20Systems%20Designer">Michael</a> and <a href="https://www.cfr.org/bio/will-merrow">Will</a> for their mentorship.
  </p>
  <h2>Features</h2>
  <ul>
  <li><strong>Scroll-driven interaction & animations</strong></li>
  <li><strong>Dynamic stacked-bar transitions</strong></li>
  <li><strong>Responsive, mobile-friendly layout</strong></li>
  <li><strong>Wrapped annotations</strong></li>
  <li><strong>Persistent chart display</strong></li>
  </ul>

  <h2>Technologies Used</h2>
  <ul>
    <li>D3.js for data visualization</li>
    <li>Intersection Observer API for scroll tracking</li>
    <li>CSS Flexbox for responsive layout</li>
    <li>SVG for vector graphics rendering</li>
  </ul>
</section>

<!-- Scroll container - Main visualization container -->
<section class="text-content">
<h1 class="article-title">Guac Shock: Understanding Tariffs With Avocados</h1>
<p class="article-description"><em>The majority of avocados consumed in the United States come from Mexico. Here's how tariffs could make the price of avocados—and other goods—more expensive.</em>
</p>
<p class="article-author">
Article by Will Merrow and Andrew Huang
</p>
</section>
<section class="text-content">
<p>Tariffs are a tax on foreign-made goods. When the United States imposes tariffs on another country, it means that U.S. businesses and consumers must pay a tax to the U.S. government whenever they import products from that country. As part of the Donald Trump administration's planned tariffs, U.S. businesses and consumers will pay a tax whenever they buy goods from Canada, China, or Mexico. Let's use the price of an avocado to see how this works in practice.</p>
</section>
<section class="scroll-container">
  <div class="scroll-info">
    <div id="chart-container" class="chart">
      <!-- Scenario title displayed above chart -->
      <h1 id="scenario-title" class="scenario-title"></h1>
      <!-- Left price label - Shows total price -->
      <div id="price-label" class="price-label">
        <span class="avocado">🥑</span> Price of an avocado for U.S. consumer
        <span class="price-value">$0.00</span>
      </div>
      <!-- Chart containers for different screen sizes -->
      <div id="chart-desktop"></div>
    </div>
  </div>
</section>

<section class="text-content">
  <p>Tariffs are a tax on foreign-made goods. When the United States imposes tariffs on another country, it means that U.S. businesses and consumers must pay a tax to the U.S. government whenever they import products from that country. As part of the Donald Trump administration's planned tariffs, U.S. businesses and consumers will pay a tax whenever they buy goods from Canada, China, or Mexico. Let's use the price of an avocado to see how this works in practice.</p>
  
  <p>This is a highly simplified example, as the supply chains of most goods are far more complex. The price of an avocado after tariffs would also depend on decisions made by shipping companies, wholesalers and other intermediaries, and the Mexican supplier. More complex products such as cars involve importing many components—some of which cross the border multiple times, meaning the net price increase from a 25 percent tariff could end up being significantly larger than 25 percent.</p>
  
  <p>And this is not just a hypothetical. The vast majority of avocados eaten by U.S. citizens come from Mexico, including an estimated $2.7 billion worth of avocados imported in 2024. In fact, free trade with Mexico is partly responsible for the rise of avocados in the U.S. diet.</p>
  
  <p>Blanket tariffs on Canada, China, and Mexico would also result in higher prices for many different products, not just groceries. Everything from cars and crude oil to smartphones and computers could be significantly affected. Inflation was a core issue for many voters in the 2024 U.S. election, and price increases from tariffs could add to this concern.</p>
  
  <p>Nor does this example account for inevitable retaliation. Trade partners could choose to impose their own tariffs on the United States in response, as China did in Trump's first term, cutting off access to markets for U.S. exporters. The business these exporters lose will force them to ultimately make painful cuts to jobs, wages, and investment.</p>
  
  <p>To be sure, tariffs are not all bad. They can be a legitimate tool for countering unfair trade practices—such as the dumping of artificially cheap Chinese goods into U.S. markets—that harm U.S. industries. They can also be used to support U.S. sectors critical to national defense and economic stability, or to protect certain U.S. industries from competition. These advantages provide a rationale for the current bipartisan consensus across the Joe Biden and Trump administrations around tariff use. But what ultimately matters is how they're used—like a surgeon's scalpel, targeting specific products and minimizing economic harm, or a blunt weapon, pummeling economies across the board.</p>
  
  <p>Tariffs also generate revenue for the government, although economists generally agree that these gains are far outweighed by the economic pain tariffs cause and cannot replace U.S. income taxes as a source of government income. According to the nonpartisan Tax Foundation, the Trump administration's planned tariffs would generate some $90 billion in revenue in 2025, but lead to 330,000 fewer jobs. This pales in comparison to the some $3 trillion that the federal government received from corporate and individual income taxes in 2024.</p>
</section>

```js
/********************************************
 * DATA LOADING AND INITIALIZATION
 ********************************************/

// 1) Fetch data using FileAttachment
const text = await FileAttachment("data/sections-text.csv").csv({
  typed: true,
}); // Text Box data
const data = await FileAttachment("data/guac-price.csv").csv({ typed: true }); // Bar Chart data

// Annotation data for chart segments
const annotations = [
  {
    key: "profit",
    label: "Profit",
    description: "Kept by U.S. Grocery",
  },
  {
    key: "tariff",
    label: "Tariff",
    description: "Paid by U.S. Grocery to U.S. government",
  },
  {
    key: "cost",
    label: "Price of imported avocado",
    description: "Paid by U.S. Grocery to Mexican supplier",
  },
];

// Process chart data - Convert string values to numbers
data.forEach((d) => {
  d.cost = +d.cost;
  d.tariff = +d.tariff;
  d.profit = +d.profit;
  d.section = +d.section;
  d.total = +d.total;
});

/********************************************
 * SCROLL SECTIONS SETUP
 ********************************************/

const container = d3.select(".scroll-container");

// Create and append scroll sections based on the CSV data
const sections = container
  .selectAll(".scroll-section")
  .data(text.sort((a, b) => a.section - b.section))
  .enter()
  .append("div")
  .attr("class", "scroll-section")
  .attr("data-step", (d) => d.section);

// Add the text-section with proper data-group attribute
const textSections = sections
  .append("div")
  .attr("class", "text-section")
  .attr("data-group", (d) => d.group || `group${d.section}`); // Use group from data or create default

// Add the text-box with HTML content
textSections
  .append("div")
  .attr("class", "text-box")
  .html((d) => {
    return d.text;
  });

/********************************************
 * CHART SETUP
 ********************************************/

// 2) Set up chart dimensions and scales
const width = 170,
  height = 300;
const margin = { top: 20, right: 180, bottom: 20, left: 0 };
const keys = ["cost", "tariff", "profit", "remainder"];
const color = d3
  .scaleOrdinal()
  .domain(keys)
  .range(["#507a3a", "#d1601d", "#92a83b", "transparent"]); // Remainder is transparent

const gap = 15; //gap between annotation and chart
const warpSize = 120; //annotation textbox width

// Bar positioning
const x = d3.scaleBand().domain(["bar"]).range([0, 70]).padding(0.2);

// Height scale
const y = d3.scaleLinear().range([height - margin.bottom, margin.top]);

// Create SVG element
const svg = d3.create("svg").attr("viewBox", [0, 0, width, height]);

// Add SVG to the DOM
document.getElementById("chart-desktop").appendChild(svg.node());

// 3) Create utility function for segments
function getSegments(sectionNum) {
  // Find the data row for this section
  let row = data.find((d) => d.section === sectionNum);
  if (!row) {
    row = { cost: 0, tariff: 0, profit: 0, total: 0 };
  }

  // Create stacked data
  const series = d3.stack().keys(keys)([row]);

  // Format for easier use
  return series.map((layer) => ({
    key: layer.key,
    y0: layer[0][0],
    y1: layer[0][1],
    value: row[layer.key],
    total: row.total,
  }));
}

// 4) Set up transition settings
const DUR = 800;
const EASE = d3.easeCubicOut;
const DELAY = { segments: 0, values: 200, total: 400 };

// 5) Initialize chart groups
svg.append("g").attr("class", "bars");
svg.append("g").attr("class", "values");
svg.append("g").attr("class", "annotations");

// Get references to HTML elements
const scenarioTitle = document.getElementById("scenario-title");
const priceLabel = document.getElementById("price-label");
const priceValue = priceLabel.querySelector(".price-value");

/********************************************
 * CHART UPDATE FUNCTION
 ********************************************/

// 6) Define update function - Called when a new section is visible
function updateChart(sectionNumber) {
  // Get segments - even for section 0
  const segs = getSegments(+sectionNumber);
  const maxY = d3.max(segs, (d) => d.y1);
  y.domain([0, maxY]);

  // Special handling for section 0 - Show empty chart
  if (sectionNumber === 0) {
    // Show price label but hide price value
    priceLabel.style.opacity = 1;
    priceValue.style.opacity = 0;

    // Hide scenario title
    scenarioTitle.style.opacity = 0;

    // For section 0, animate bars to height 0
    segs.forEach((seg) => {
      seg.y0 = 0;
      seg.y1 = 0;
      seg.value = 0;
    });
  }

  // Update bar segments
  svg
    .select("g.bars")
    .selectAll("rect.segment")
    .data(segs, (d) => d.key)
    .join(
      // Enter new bars
      (enter) =>
        enter
          .append("rect")
          .attr("class", "segment")
          .attr("x", x("bar"))
          .attr("width", x.bandwidth())
          .attr("y", y(0))
          .attr("height", 0)
          .attr("fill", (d) =>
            d.key === "remainder" ? "transparent" : color(d.key)
          )
          .call((g) =>
            g
              .transition()
              .duration(DUR)
              .ease(EASE)
              .attr("y", (d) => y(d.y1))
              .attr("height", (d) => y(d.y0) - y(d.y1))
          ),
      // Update existing bars
      (update) =>
        update
          .transition()
          .duration(DUR)
          .ease(EASE)
          .attr("y", (d) => y(d.y1))
          .attr("height", (d) => y(d.y0) - y(d.y1))
          .attr("fill", (d) =>
            d.key === "remainder" ? "transparent" : color(d.key)
          ),
      // Remove old bars
      (exit) =>
        exit
          .transition()
          .duration(DUR / 2)
          .attr("y", y(0))
          .attr("height", 0)
          .remove()
    );

  // Update value labels (skip for section 0)
  svg
    .select("g.values")
    .selectAll("text.value")
    .data(
      // Filter segments that should have labels
      sectionNumber === 0
        ? []
        : segs.filter((d) => d.value > 0 && d.key !== "remainder"),
      (d) => d.key
    )
    .join(
      // Enter new labels
      (enter) =>
        enter
          .append("text")
          .attr("class", "value")
          .attr("x", x("bar") + x.bandwidth() / 2)
          .attr("y", y(0))
          .attr("fill", "white")
          .attr("font-size", "10px")
          .attr("text-anchor", "middle")
          .attr("opacity", 0)
          .text((d) => `$${d.value.toFixed(2)}`)
          .call((g) =>
            g
              .transition()
              .delay(DELAY.values)
              .duration(DUR)
              .ease(EASE)
              .attr("y", (d) => (y(d.y0) + y(d.y1)) / 2 + 2)
              .attr("opacity", 1)
          ),
      // Update existing labels
      (update) =>
        update
          .transition()
          .delay(DELAY.values)
          .duration(DUR)
          .ease(EASE)
          .attr("y", (d) => (y(d.y0) + y(d.y1)) / 2 + 2)
          .attr("opacity", 1)
          .text((d) => `$${d.value.toFixed(2)}`),
      // Remove old labels
      (exit) =>
        exit
          .transition()
          .duration(DUR / 2)
          .attr("opacity", 0)
          .remove()
    );

  // Utility function to wrap text for annotations
  function wrapText(text, width) {
    const words = text.split(/\s+/).reverse();
    let word;
    let line = [];
    const lines = [];
    while ((word = words.pop())) {
      line.push(word);
      const testLine = line.join(" ");
      const testWidth = testLine.length * 6; // Approximate width calculation
      if (testWidth > width && line.length > 1) {
        line.pop();
        lines.push(line.join(" "));
        line = [word];
      }
    }
    lines.push(line.join(" "));
    return lines;
  }

  // Update annotations (skip for section 0)
  svg
    .select("g.annotations")
    .selectAll("text.annotation")
    .data(
      // Filter segments that should have annotations
      sectionNumber === 0
        ? []
        : segs.filter((d) => d.value > 0 && d.key !== "remainder"),
      (d) => d.key
    )
    .join(
      (enter) => {
        const annotationText = enter
          .append("text")
          .attr("class", "annotation")
          .attr("x", x("bar") + x.bandwidth() / 2) // Position next to the value
          .attr("y", (d) => (y(d.y0) + y(d.y1)) / 2 - 10) // Align with value text
          .attr("fill", (d) => color(d.key))
          .attr("font-size", "10px")
          .attr("text-anchor", "start")
          .attr("opacity", 0);

        annotationText
          .selectAll("tspan.label")
          .data((d) => {
            const annotation = annotations.find((a) => a.key === d.key);
            return annotation ? wrapText(annotation.label, warpSize) : [];
          })
          .join("tspan")
          .attr("class", "label")
          .attr("x", x("bar") + x.bandwidth() + gap)
          .attr("dy", "1.2em")
          .text((d) => d);

        // Add description with line break and styled as italic
        annotationText.each(function (d) {
          const annotation = annotations.find((a) => a.key === d.key);
          if (annotation && annotation.description) {
            const lines = wrapText(annotation.description, warpSize);
            const node = d3.select(this);

            lines.forEach((line, i) => {
              node
                .append("tspan")
                .attr("class", "description")
                .attr("x", x("bar") + x.bandwidth() + gap)
                .attr("dy", i === 0 ? "1.5em" : "1.2em") // larger spacing for first line
                .attr("font-style", "italic")
                .attr("font-size", "8px")
                .attr("fill-opacity", 0.8)
                .text(line);
            });
          }
        });

        annotationText.call((g) =>
          g
            .transition()
            .delay(DELAY.values)
            .duration(DUR)
            .ease(EASE)
            .attr("opacity", 1)
        );
      },
      (update) => {
        // First, remove all existing tspans to avoid duplicates
        update.selectAll("tspan").remove();

        // Re-add label tspans
        update
          .selectAll("tspan.label")
          .data((d) => {
            const annotation = annotations.find((a) => a.key === d.key);
            return annotation ? wrapText(annotation.label, warpSize) : [];
          })
          .join("tspan")
          .attr("class", "label")
          .attr("x", x("bar") + x.bandwidth() + gap)
          .attr("dy", "1.2em")
          .text((d) => d);

        // Re-add description tspans
        update.each(function (d) {
          const annotation = annotations.find((a) => a.key === d.key);
          if (annotation && annotation.description) {
            const lines = wrapText(annotation.description, warpSize);
            const node = d3.select(this);

            lines.forEach((line, i) => {
              node
                .append("tspan")
                .attr("class", "description")
                .attr("x", x("bar") + x.bandwidth() + gap)
                .attr("dy", i === 0 ? "1.5em" : "1.2em")
                .attr("font-style", "italic")
                .attr("font-size", "8px")
                .attr("fill-opacity", 0.8)
                .text(line);
            });
          }
        });

        update
          .transition()
          .delay(DELAY.values)
          .duration(DUR)
          .ease(EASE)
          .attr("y", (d) => (y(d.y0) + y(d.y1)) / 2 - 10)
          .attr("opacity", 1)
          .attr("fill", (d) => color(d.key));
      },
      (exit) =>
        exit
          .transition()
          .duration(DUR / 2)
          .attr("opacity", 0)
          .remove()
    );

  // Update the HTML scenario title
  const sectionText = text.find((t) => t.section === +sectionNumber);

  // Update title with transition
  if (sectionNumber >= 2 && sectionText && sectionText.title) {
    // Only transition if the title is different
    if (scenarioTitle.textContent !== sectionText.title) {
      // First fade out
      scenarioTitle.style.opacity = 0;

      // Then change text and fade in after transition completes
      setTimeout(() => {
        scenarioTitle.textContent = sectionText.title;
        scenarioTitle.style.opacity = 1;
      }, 300);
    } else {
      // If same title, just ensure it's visible
      scenarioTitle.style.opacity = 1;
    }
  } else {
    // Hide title if no title for this section
    scenarioTitle.style.opacity = 0;
  }

  // Update the HTML price label (already handled for section 0)
  if (sectionNumber !== 0) {
    const topSeg = segs[segs.length - 1];

    // Always show price label for all sections
    priceLabel.style.opacity = 1;

    // Fill price value and control its visibility
    if (topSeg.total > 0) {
      priceValue.textContent = `$${topSeg.total.toFixed(2)}`;
      priceValue.style.opacity = 1;
    } else {
      priceValue.style.opacity = 0;
    }
  }
}

/********************************************
 * INTERSECTION OBSERVER SETUP
 ********************************************/

// 7) Set up the intersection observer
const secs = document.querySelectorAll(".scroll-section");
let lastVisibleSection = 0; // Keep track of the last visible section

const obs = new IntersectionObserver(
  (ents) => {
    // Find the most visible section
    let best = ents
      .filter((e) => e.isIntersecting)
      .sort((a, b) => b.intersectionRatio - a.intersectionRatio)[0];

    if (best) {
      // If a section is visible, update the chart
      const step = +best.target.dataset.step;
      lastVisibleSection = step; // Update the last visible section
      updateChart(step);
    } else if (
      window.scrollY >
      document.body.scrollHeight - window.innerHeight - 100
    ) {
      // Last chart persist : If we're near the bottom of the page, keep showing the last section
      updateChart(lastVisibleSection);
    } else if (window.scrollY < 100) {
      // If we're near the top of the page, show the first section
      updateChart(0);
    } else {
      // Otherwise, continue showing last visible section
      updateChart(lastVisibleSection);
    }
  },
  {
    rootMargin: "0px", //"-50% 0% -50% 0%", //Consider middle 50% of viewport(top, right, bottom, left).
    threshold: 0.1, //[0, 0.25, 0.5, 0.75, 1], // Various thresholds for better detection
  }
);

// 8) Start observing sections and initialize chart
secs.forEach((s) => obs.observe(s));

// Initialize with section 0 state - no chart, only price label
updateChart(0);

// Add scroll event listener to handle cases where no section is visible
window.addEventListener("scroll", () => {
  // Check if any section is currently intersecting
  const anyIntersecting = Array.from(secs).some((section) => {
    const rect = section.getBoundingClientRect();
    // Consider a section intersecting if it's in or near the viewport
    return rect.top < window.innerHeight && rect.bottom > 0;
  });

  // If we're near the bottom of the page and no section is intersecting
  if (
    window.scrollY > document.body.scrollHeight - window.innerHeight - 200 &&
    lastVisibleSection > 0
  ) {
    updateChart(lastVisibleSection);
  } else if (window.scrollY < 300 || !anyIntersecting) {
    // If we're at the top of the page or no sections are visible, show section 0
    updateChart(0);
  }
});

// Cleanup on invalidation
invalidation.then(() => {
  obs.disconnect();
  window.removeEventListener("scroll");
});
```

<style>
/********************************************
 * PROJECT DESCRIPTION
 ********************************************/
.project-description {
  max-width: 700px;
  margin: 2rem auto 4rem auto;
  padding: 2rem;
  background-color: #f8f8f2;
  border-left: 5px solid #507a3a;
  border-radius: 4px;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
  font-family: var(--sans-serif);
}

.project-description h1 {
  color: #507a3a;
  font-size: 32px;
  margin-bottom: 1rem;
  border-bottom: 2px solid #d1601d;
  padding-bottom: 0.5rem;
}

.project-description h2 {
  color: #d1601d;
  font-size: 22px;
  margin-top: 2rem;
  margin-bottom: 1rem;
}

.project-description p {
  line-height: 1.6;
  font-size: 16px;
}

.project-description ul {
  list-style-type: none;
  padding-left: 0;
}

.project-description li {
  position: relative;
  padding-left: 1.5rem;
  margin-bottom: 0.5rem;
  line-height: 1.5;
}

.project-description li:before {
  content: "🥑";
  position: absolute;
  left: 0;
  font-size: 0.9rem;
}

.project-description strong {
  color: #507a3a;
}

/********************************************
 * TEXT CONTENTS
 ********************************************/
 .text-content {
    max-width: 720px;
    margin: 0 auto;
    margin-bottom: 40px;
    font-size 10pt;
letter-spacing 0.32px;
line-height 1.3em;
}

.article-title {
    font-size: 46px;
font-weight: 400;
letter-spacing: 0.32px;
font-family: var(--sans-serif);
}

.article-description {
    font-size: 14px;
letter-spacing: 0.32px;
line-height: 21.98px;
}

.article-author {
    font-size: 16px;
letter-spacing: 0.32px;
font-family: var(--sans-serif);
}
/********************************************
 * LAYOUT AND CONTAINERS
 ********************************************/
:root {
  --text-width: 50%;  /* Text container width */
  --chart-width: 50%; /* Chart container width */
  --sans-serif: Larsseit, Arial, serif;
  --serif: "Lyon Text Web", serif;
}
* {
    color: rgb(65, 44, 38);
}
/* Container for the whole scrolling experience */
.scroll-container {
  position: relative;
  margin: 1rem auto;
  font-family: var(--sans-serif);
  max-width: 900px;
}

.scroll-info {
  position: sticky;
  height: 100vh;
  top: 0;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: end;
  font-size: 64px;
}

/* Section for each step of the scroll */
.scroll-section {
  position: relative;
  height: 100vh;
  display: flex;
  align-items: start;
  justify-content: center;
  padding: 1rem;
  box-sizing: border-box;
}

/* Extra space after the last section */
.scroll-section:last-child {
  margin-bottom: 50vh; /* Add extra space at the bottom */
  position: relative;
}

/* Dummy element to trigger observer at bottom */
.scroll-container::after {
  content: "";
  display: block;
  height: 100px;
  width: 100%;
}

/********************************************
 * CHART ELEMENTS
 ********************************************/

/* Scenario title styling */
.scenario-title {
  position: absolute;
  top: 0;
  left: 0;
  text-align: left;
  font-size: 1rem;
  color: #4b535d;
  opacity: 0;
  transition: opacity 0.3s ease;
  margin: 0;
}

/* Price label styling */
.avocado {
  font-size: 30pt;
  line-height: 1.5em;
}

.price-label {
  width: 30%;
  line-height: 1.3em;
  font-size: 12pt;
  text-align: right;
  display: flex;
  flex-direction: column;
  opacity: 0;
  transition: opacity 0.8s ease;
  z-index: 10;
  min-width: 100px;
  max-width: 118px;
}

/* Price value display */
.price-value {
  font-size: 15pt;
  font-weight: bold;
  margin-top: 0.5em;
  height: 1.2em; /* Fixed height to prevent layout shift */
  opacity: 0; /* Start with opacity 0 */
  transition: opacity 0.8s ease; /* Smooth transition */
}

/* Chart container element */
#chart-container {
  position: relative;
  z-index: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding-top: 2rem;
  gap: 1rem;
}

/********************************************
 * TEXT STYLES AND HIGHLIGHTING
 ********************************************/

.text-box {
  background: white;
  padding: 20px;
  max-width: 300px;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  opacity: 0.93;
  position: relative;
  margin: 0 20px;
  font-family: "Lyon-RegularNo2", serif;
  color: #4b535d;
  letter-spacing: 0.36px;
  font-size: 16px;
}

/* Highlighting text elements */
.profit {
  background-color: #92a83b; 
  color: #ffffff;
  padding: 2px 4px;
  border-radius: 2px;
}

.price {
  font-weight: bold;
}

/* Color class styles */
.text-avocado { 
  background-color: #507a3a; 
  color: #ffffff; 
}

.text-tariff { 
  background-color: #d1601d; 
  color: #ffffff; 
}

.text-profit { 
  background-color: #92a83b; 
  color: #ffffff; 
}

.text-consumer { 
  background-color: #4b535d; 
  color: #ffffff; 
  font-family: "Lyon-RegularNo2" !important;
  letter-spacing: 0.36px;
  font-size: 16px;
}

/* Annotation Styles */
.annotation, .values {
  font-weight: 600;
}

.description {
  font-style: italic;
  font-size: 0.8em;
  font-weight: 500;
}

/********************************************
 * RESPONSIVE DESIGN
 ********************************************/

/* Desktop */
@media (min-width: 760px) {
  .project-description {
    padding: 2.5rem;
  }
  
  #chart-desktop {
    display: flex;
    height: 500px;
  }
  
  #chart-container {
    width: 50%;
  }
  
  .text-section {
    position: absolute;
    top: 0;
    left: 0;
    display: flex;
    justify-content: center;
  }
}

/* Tablet and Mobile */
@media (max-width: 759px) {
  .project-description {
    padding: 1.5rem;
    margin: 1rem auto 2rem auto;
  }
  
  .project-description h1 {
    font-size: 24px;
  }
  
  .project-description h2 {
    font-size: 18px;
  }
  
  .scenario-title {
    text-align: center;
    width: 100%;
  }
  
  #chart-container {
    width: 100%;
    max-width: 500px;
    margin: 0 auto;
  }
  
  .text-section {
    margin: 20px auto;
  }
  
  .text-box {
    margin: 20px auto;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1) !important;
  }
  
  #chart-desktop {
    display: flex;
    justify-content: end;
    width: 70%;
    max-width: 250px;
  }
}
</style>
