# Power calculator: instructor notes

A single-page tool for exploring how sample size, effect size and power trade off in a two-group study, and for drawing a simulated sample to see where its average lands. Everything lives in `index.html`; there are no other files to deploy and nothing is sent to a server.

These notes collect the technical details that the page itself deliberately leaves out.

## Publishing on GitHub Pages

1. Create a public repository and upload `index.html` (and this README, if you like) to the top level.
2. In the repository, open **Settings → Pages**, choose **Deploy from a branch**, and select `main` and `/ (root)`.
3. After a minute or so the site is live at `https://<username>.github.io/<repository>/`.

The headings use Source Serif 4 from Google Fonts; if it can't load, the page falls back to Charter or Georgia.

## What the calculator computes

**The test.** Power is calculated for a two-sample *t*-test with pooled variance (equal standard deviations assumed), two-sided by default, at the significance level entered. "Difference in means" is treatment minus control, in the measurement's own units; "standard deviation" is the common within-group SD.

**The method.** Power comes from the noncentral *t* distribution with *n*₁ + *n*₂ − 2 degrees of freedom and noncentrality Δ / (σ√(1/*n*₁ + 1/*n*₂)), counting both rejection tails for a two-sided test. That is the same calculation as G\*Power, or R's `power.t.test(..., strict = TRUE)`. It is computed by numerically integrating over the distribution of the sample SD, so it stays exact for any group size rather than relying on a normal approximation.

**Solving.**
- *Sample size* returns the smallest whole number of people reaching the target power. "Both" keeps the groups equal; "Control only" or "Treatment only" grows one group while the other stays fixed.
- Growing only one group has a ceiling: as that group becomes infinite, the standard error can't drop below σ/√(fixed group size). When the target is above that ceiling, the page says the target is not reachable and suggests enlarging the other group.
- *Difference* returns the smallest true difference detectable at the target power, rounded **up** to four significant figures so the displayed value still meets the target.
- Sample-size searches stop at 10,000,000 per group.

**Verification.** Power values were checked against SciPy's `nct` distribution on 378 random designs (group sizes 2 to 100,000, α from 0.0001 to 0.1, one- and two-sided); the largest discrepancy was about 2 × 10⁻¹⁰. The sample-size and difference solvers matched brute-force SciPy searches in every one of 120 test cases, including 13 unreachable one-group designs. The textbook check holds: *d* = 0.5, 80% power, two-sided α = 0.05 gives 64 per group.

## What the Sample button does

**The data.** Each press draws a fresh sample: *n*₁ control values from a normal distribution with the "control group average" and the SD entered, and *n*₂ treatment values from a normal distribution shifted by the difference in means. The control average only positions the axis; it has no effect on power.

**The histogram.** Each bar is the share of its group falling in that bin, so groups of different sizes sit on a common scale; control and treatment bars are side by side. The vertical scale is set so the tallest bar or curve reaches about 82% of the plot height. Bin width follows the Freedman–Diaconis rule for normal data, kept between about 9 and 70 bins and rounded to a readable step, and stays fixed for a given design so that repeated samples share the same axis. Values more than 4 SDs outside the two population means fall off the plot (they are still included in the averages); for typical group sizes this almost never happens.

**The curves.** The two curves are the population distributions the sample is drawn from, drawn on the same scale as the bars (the expected share of a group per bin: bin width × normal density):
- **Blue (no real difference):** normal, with the control group average and the SD entered. If there is no real difference, both groups' bars should follow it.
- **Orange (the difference is real):** the same curve shifted by the difference entered. If the difference is real, the treatment group's bars should follow it, while the control group's still follow blue.

The curves are set by the inputs, not fitted to the sample, so they don't change between samples or with group size. Each curve's peak is where that group's average is expected to land. What changes with group size is how closely the bars fill out the curves and how far the averages wander from the peaks from one sample to the next. The lines marking the two sample averages are drawn last, on top of everything, and labeled with their values.

**What the picture leaves out.** Because the curves describe individuals, their overlap reflects the difference relative to the SD, not the power. At 64 per group with a difference of half an SD, the curves overlap heavily even though power is 80%. The precision of the averages, which is what power depends on, isn't drawn; it shows up as how much the average lines jump when **Sample** is pressed repeatedly. It may be worth saying this explicitly, so that students don't read the overlap of the curves as the chance of detecting the difference.

**Changing settings.** After a change to the group sizes, difference, SD or control average, the old sample dims until **Sample** is pressed again. Large samples are drawn in chunks with a progress indicator; 2 million per group took about half a second in testing.

## Other behaviour worth knowing

- **Defaults.** The page opens solving for power, with 20 people per group, a difference of 5, an SD of 5 and a significance level of 0.01, which gives 67.3% power. The defaults are the `value` attributes of the inputs in `index.html` (search for `id="sd"`, `id="alpha"` and so on), plus `solve: 'power'` near the top of the script; a link with parameters overrides them.
- **Switching modes keeps the design.** Changing what is solved for leaves the current numbers in place and only changes which field is computed. For example, solving for sample size and then switching to "Power" shows the power of that same design. Rounded values in solved fields are tracked exactly behind the scenes, so switching back and forth doesn't drift.
- **Links.** The address bar always reflects the current inputs, and **Copy link** copies it, so you can post a link that opens a specific scenario.

## Link parameters

| Parameter | Meaning | Example |
|---|---|---|
| `solve` | What to solve for: `n`, `diff` or `power` | `solve=power` |
| `grow` | When solving for sample size: `both`, `control` or `treatment` | `grow=both` |
| `n1`, `n2` | Control and treatment group sizes | `n1=20&n2=20` |
| `diff` | Difference in means (treatment − control) | `diff=5` |
| `sd` | Standard deviation | `sd=10` |
| `power` | Target power, in percent | `power=80` |
| `alpha` | Significance level | `alpha=0.05` |
| `mu` | Control group average used for sampling | `mu=100` |
| `sides` | `sides=1` switches to a one-sided test (in the direction of the difference entered). There is no on-page control for this. | `sides=1` |

Example: `index.html?solve=power&n1=10&n2=10&diff=5&sd=10&alpha=0.05&mu=70` opens the power for 10 per group, with samples centered at 70.

## Things to try in class

- Start from the defaults (20 per group, difference 5, SD 5, significance 0.01; power 67.3%), press **Sample** a few times, then shrink the difference or loosen the significance level to 0.05 and watch the power change. For a noisier picture, drop to 10 per group with an SD of 10 and press **Sample** several times. The bars only loosely follow the curves, and the averages jump around; sometimes the treatment average lands below the control average even though the effect is real. Then try 64 and 500 per group: the bars fill out the curves and the averages settle near the peaks.
- Point out that the two curves overlap heavily even at 80% power. Individuals from the two groups are hard to tell apart, yet with enough people the averages can be.
- Hold the sample size fixed and raise the SD: the curves widen and overlap more, the averages wander more, and power falls, even though the difference hasn't changed.
- Fix the treatment group at 30 and choose "Control only" to show that adding people to one group can't make up for a small other group.
