
<!-- README.md is generated from README.Rmd. Please edit that file -->
rogme
=====

Robust Graphical Methods For Group Comparisons

**DEVELOPMENT VERSION: not beta tested yet, functions not fully documented...**

The `rogme` R package provides graphical tools to compare groups of continous observations. The goal is to illustrate and quantify how and by how much groups differ. The current version of the package is limited to comparing two groups. Future developments will extend the tools to deal with multiple groups and interactions.

The package can be installed using these commands:

``` r
install.packages("devtools")
devtools::install_github("GRousselet/rogme")
```

The approach behind the package is described here:

[A few simple steps to improve the description of group results in neuroscience](http://onlinelibrary.wiley.com/doi/10.1111/ejn.13400/full)

[Modern graphical methods to compare two groups of observations](https://figshare.com/articles/Modern_graphical_methods_to_compare_two_groups_of_observations/4055970)

The second reference contains extensive examples using `rogme`.

`rogme` uses `ggplot2` for graphical representations, and the main statistical functions were developed by Rand Wilcox, as part of his [`WRS`](https://dornsife.usc.edu/labs/rwilcox/software/) package.

The main tool in `rogme` is the [shift function](https://garstats.wordpress.com/2016/07/12/shift-function/). A shift function shows the difference between the quantiles of two groups as a function of the quantiles of one group. For inferences, the function returns an uncertainty interval for each quantile difference. By default, the deciles are used. Currently, confidence intervals are computed using one of two percentile bootstrap techniques. Highest density intervals and [Bayesian bootstrap](https://github.com/rasmusab/bayesboot) intervals will be available eventually.

Functions
---------

All the functions rely on the [Harrell-Davis quantile estimator](https://garstats.wordpress.com/2016/06/09/the-harrell-davis-quantile-estimator/), computed by the `hd()` function.

### Shift function

In the `WRS` package, the shift function can be calculated using:

-   `shifthd()` or `qcomhd()` for independent groups
-   `shiftdhd()` or `Dqcomhd()` for dependent groups

These functions can also produce non-ggplot figures.

In `rogme`, the shift function can be calculated using:

-   `shifthd()` or `shifthd_pbci()` for independent groups
-   `shiftdhd()` or `shiftdhd_pbci()` for dependent groups

Illustrations of the results is handled separately by `plot_sf()` and `plot_pbsf()`.

You can see the shift function in action [here](https://garstats.wordpress.com/2016/07/12/shift-function/) and [here](http://onlinelibrary.wiley.com/doi/10.1111/ejn.13100/full).

### Difference asymmetry function

The difference asymmetry function is another powerful graphical and inferential tool. In the `WRS` package it is calculated using:

-   `qwmwhd()` for independent groups
-   `difQpci()` for dependent groups

In `rogme`, these functions have been renamed:

-   `asymhd()` for independent groups
-   `asymdhd()` for dependent groups
-   `plot_diff_asym()` to plot the results

You can see the difference asymmetry function in action [here](https://garstats.wordpress.com/2016/07/19/typical-differences/).

Shift function demo
-------------------

Detailed illustration of the shift function using two distributions that differ in spread.

``` r
# generate data
set.seed(21)
g1 <- rnorm(1000) + 6
g2 <- rnorm(1000) * 1.5 + 6

# make tibble
library(rogme)
#> Loading required package: ggplot2
df <- mkt2(g1, g2)
```

First, we generate the 1D scatterplots for the two groups.

``` r
#> scatterplots alone
ps <- plot_scat2(df,
                 xlabel = "",
                 ylabel = "Scores (a.u.)",
                 alpha = 1,
                 shape = 21,
                 colour = "grey10",
                 fill = "grey90") # scatterplots
ps <- ps + coord_flip()
ps
```

![](README-files/README-unnamed-chunk-4-1.png)

Second, we compute the shift function and then plot it.

``` r
#> compute shift function
sf <- shifthd(data = df, formula = obs ~ gr, nboot = 200)

#> plot shift function
psf <- plot_sf(sf, plot_theme = 2)

#> change axis labels
psf <- psf +
  labs(x = "Group 1 quantiles of scores (a.u.)",
       y = "Group 1 - group 2 \nquantile differences (a.u.)")

#> add labels for deciles 1 & 9
psf <- add_sf_lab(psf, sf, y_lab_nudge = .1)
psf
```

![](README-files/README-unnamed-chunk-5-1.png)

Third, we make 1D scatterplots with deciles and colour coded differences.

``` r
p <- plot_scat2(df,
                xlabel = "",
                ylabel = "Scores (a.u.)",
                alpha = .3,
                shape = 21,
                colour = "grey10",
                fill = "grey90") # scatterplots
p <- plot_dec_links(p, sf,
                    dec_size = 1,
                    md_size = 1.5,
                    add_rect = TRUE,
                    rect_alpha = 0.1,
                    rect_col = "grey50",
                    add_lab = TRUE) # superimposed deciles + rectangle
p <- p + coord_flip() # flip axes
p
```

![](README-files/README-unnamed-chunk-6-1.png)

Finally, we combine the three plots into one figure.

``` r
library(cowplot)
cowplot::plot_grid(ps, p, psf, labels=c("A", "B", "C"), ncol = 1, nrow = 3,
                   rel_heights = c(1, 1, 1), label_size = 20, hjust = -0.5, scale=.95)
```

![](README-files/README-unnamed-chunk-7-1.png)

**Panel A** illustrates two distributions, both n = 1000, that differ in spread. The observations in the scatterplots were jittered based on their local density, as implemented in `ggbeeswarm::geom_quasirandom`.

**Panel B** illustrates the same data from panel A. The dark vertical lines mark the deciles of the distributions. The thicker vertical line in each distribution is the median. Between distributions, the matching deciles are joined by coloured lined. If the decile difference between group 1 and group 2 is positive, the line is orange; if it is negative, the line is purple. The values of the differences for deciles 1 and 9 are indicated in the superimposed labels.

**Panel C** focuses on the portion of the x-axis marked by the grey shaded area at the bottom of panel B. It shows the deciles of group 1 on the x-axis – the same values that are shown for group 1 in panel B. The y-axis shows the differences between deciles: the difference is large and positive for decile 1; it then progressively decreases to reach almost zero for decile 5 (the median); it becomes progressively more negative for higher deciles. Thus, for each decile the shift function illustrates by how much one distribution needs to be shifted to match another one. In our example, we illustrate by how much we need to shift deciles from group 2 to match deciles from group 1.

More generally, a shift function shows quantile differences as a function of quantiles in one group. It estimates how and by how much two distributions differ. It is thus a powerful alternative to the traditional t-test on means, which focuses on only one, non-robust, quantity. Quantiles are robust, intuitive and informative.
