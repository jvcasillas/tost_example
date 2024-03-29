
# TOSTs with `TOSTER`

In linguistic research we often test hypotheses that intend to show that
different means are statistically different from each other. For
example, if we want to compare groups with different linguistic
experience on some metric (e.g., VOT in stop production). We might
hypothesize that a group of native English L2 learners of Spanish will
produce /t/ with higher VOT when compared with a group of native Spanish
speakers. This hypothesis is not particularly interesting, as there are
decades of investigations showing English speakers produce voiceless
stops with long-lag VOT and Spanish speakers produce them with short-lag
VOT.

Sometimes, however, our research questions may not seek out differences
between groups, but rather we want to test to see if they are *not*
different. Imagine, for example, that we wanted to compare two groups of
learners on some metric, but first we needed to be certain that they
were comparable in terms of working memory. In this case, before we test
our hypothesis of interest we would want to assess the working memory of
both groups and demonstrate that one group does not have better working
memory than the other. To continue with our VOT example, we might be
interested in testing whether or not a group of native English L2
learners of Spanish with extremely high Spanish proficiency has acquired
a native-like phonetic category for Spanish /t/. In this case, we might
be interested in a test showing that the learners’ VOT for /t/ is not
statistically different from that of the native Spanish speaker group.

For cases like these, we can use two one-sided tests (TOSTs, or tests of
equivalence). In order to conduct a TOST, we need to consider what it
means to be statistically non-different. In a null-hypothesis
significance testing (NHST) framework, we generally test to see if a
value is different from a point null (usually 0). If we stop to think
about this, it is rarely the case that two means are exactly equivalent.
For example, in speech research we deal with what is called the lack of
invariance problem, which, in short, refers to the fact that we never
produce two utterances in *exactly* the same way, physically,
acoustically. So if I were to say “dog” five times, the /d/ will never
be exactly the same. For this reason, we need to establish equivalence
bounds (these are similar to smallest effect size of interest \[SESI\]
and regions of practical equivalence \[ROPEs\], if you are familiar with
them). That is, we need to think about how small a difference is small
enough, in our expert opinion, to be considered equivalent to a null
effect (i.e., 0).

At first glance this is much more difficult than testing for a
difference, e.g., a t-test, because we are required to think a lot more
about what we know about the phenomenon in question. For example, if you
are doing research using reaction times, you might find a difference
between groups of 3 ms. But can a 3 ms difference actually mean anything
cognitively? Continuing with our VOT example, we know that monolingual
English speakers generally produce English /t/ with VOT values between
30 and 60 ms, and that Spanish monolinguals usually produce Spanish /t/
with VOT between 0 and 25 ms, and there can certainly be some overlap.
These are important factors one should take into consideration when
conducting a TOST. In other words, domain expertise is often essential.

In what follows we will work through an example that considers the case
described above. We are interested in a group of highly proficient L2
learners of Spanish. We want to see how native-like they are. We will
assume that they have spent many years living in Mexico and we
hypothesize that their linguistic experience has led them to acquire a
new phonetic category for Spanish /t/. We will conduct a TOST to see if
their /t/ is statistically equivalent to that of a group of monolingual
Spanish speakers also living in Mexico. Given what we know about VOT, we
must establish equivalence bounds. We believe that a group difference of
± 5 ms can be considered statistically equivalent. Note: one cannot
simply make up this value, but rather it needs to be motivated
theoretically and established *a priori*, ideally before you even
collect your data. What the TOST will do is conduct two one-sided tests
to statistically reject effects ≤ -5 and ≥ 5 ms.

We will use the package `TOSTER` for our analysis and the `tidyverse` to
plotting and general tidying. If you don’t have these packages you must
install them if you would like to reproduce the code in this tutorial.

``` r
library("tidyverse")
library("TOSTER")
```

Now we will simulate some VOT data.

``` r
# Set seed for reproducibilitu
set.seed(20191208)

# Number of participants per group
n <- 40

# Generate dataframe
vot_data <- 
  tibble(
    participant = c(glue::glue("mono", "{num}", num = 1:n), 
                    glue::glue("l2", "{num}", num = 1:n)), 
    group = gl(n * 2, n, length = n * 2, labels = c("Monolingual", "L2")), 
    phon = "t", 
    vot = c(rnorm(n, 15, 6), rnorm(n, 17, 7))
  )

# Calculate group means
grp_means <- vot_data %>% 
  group_by(group) %>% 
  summarize(mean_t = mean(vot), sd_t = sd(vot)) %>% 
  mutate_if(is.numeric, round, digits = 2)
grp_means
```

    ## # A tibble: 2 x 3
    ##   group       mean_t  sd_t
    ##   <fct>        <dbl> <dbl>
    ## 1 Monolingual   15.8  5.75
    ## 2 L2            17.6  6.57

We have simulated a data set with 80 participants (40 L2 learners, 40
monolingual Spanish speakers). For each participant we have their
average VOT value for /t/. We plot the individual and group means below.

``` r
vot_data %>% 
  ggplot(., aes(x = group, y = vot)) + 
    geom_hline(yintercept = 30, lty = 3) + 
    geom_jitter(width = 0.2, alpha = 0.4, aes(color = group), show.legend = F) + 
    stat_summary(fun.data = mean_cl_boot, geom = "pointrange", pch = 21, 
                 size = 1.5, aes(fill = group), show.legend = F) + 
    ylim(-5, 40) + 
    scale_color_brewer(palette = "Set1", name = NULL) + 
    scale_fill_brewer(palette = "Set1", name = NULL) + 
    labs(y = "VOT (ms)", x = NULL, caption = "Mean ± 95% CI", 
         title = "Voice-onset time", 
         subtitle = "VOT of /t/ for monolingual speakers and L2 learners") +
    coord_flip() + 
    theme_minimal(base_family = "Times", base_size = 12)
```

<img src="README_files/figure-gfm/basic-plot-1.png" width="100%" />

The group mean for the monolingual Spanish speakers is 15.77 ± 5.75 SD
and for the L2 learners it is 17.56 ± 6.57 SD. We have marked with a
vertical dotted line the 30 ms point to remind us where researchers
typically distinguish between short-lag and long-lag VOT values. We can
see that the L2 learners are producing a Spanish /t/ well below the 30
ms line, indicating that their VOT is indeed native-like. Now we want to
know if it is statistically non-different from the native speakers. Some
people erroneously test this hypothesis by conducting a t-test. If the
p-value is above 0.05 then they might *incorrectly* conclude that the
groups produce VOT the same, i.e., that there is no difference. Let’s do
this to see what value we get.

``` r
bad_t_test <- t.test(vot ~ group, data = vot_data, paired = F, var.equal = T)
print(bad_t_test)
```

    ## 
    ##  Two Sample t-test
    ## 
    ## data:  vot by group
    ## t = -1.2969, df = 78, p-value = 0.1985
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -4.5369912  0.9577074
    ## sample estimates:
    ## mean in group Monolingual          mean in group L2 
    ##                  15.77152                  17.56116

The t-test shows no difference between the groups, but we know that the
“absence of evidence is not evidence of absence”, thus we cannot
conclude that the difference between the VOT of the L2 and native groups
is equivalent. Let’s conduct a TOST. We will use the mean and SD values
reported above. Recall that we set our equivalence bounds at ± 5 ms.

``` r
# Store means, sds and equivalence bounds as objects
l2_mean <- filter(grp_means, group == "L2") %>% pull(mean_t)
l2_sd   <- filter(grp_means, group == "L2") %>% pull(sd_t)
l2_n    <- n
mo_mean <- filter(grp_means, group == "Monolingual") %>% pull(mean_t)
mo_sd   <- filter(grp_means, group == "Monolingual") %>% pull(sd_t)
mo_n    <- n
lb      <- -5
ub      <-  5

# Conduct tost
TOSTtwo.raw(
  m1 = l2_mean, sd1 = l2_sd, n1 = l2_n, # learners
  m2 = mo_mean, sd2 = mo_sd, n2 = mo_n, # monolinguals
  low_eqbound = lb,
  high_eqbound = ub,
  alpha = 0.05, 
  var.equal = TRUE)
```

    ## TOST results:
    ## t-value lower bound: 4.92    p-value lower bound: 0.000002
    ## t-value upper bound: -2.33   p-value upper bound: 0.011
    ## degrees of freedom : 78
    ## 
    ## Equivalence bounds (raw scores):
    ## low eqbound: -5 
    ## high eqbound: 5
    ## 
    ## TOST confidence interval:
    ## lower bound 90% CI: -0.508
    ## upper bound 90% CI:  4.088
    ## 
    ## NHST confidence interval:
    ## lower bound 95% CI: -0.958
    ## upper bound 95% CI:  4.538
    ## 
    ## Equivalence Test Result:
    ## The equivalence test was significant, t(78) = -2.325, p = 0.0113, given equivalence bounds of -5.000 and 5.000 (on a raw scale) and an alpha of 0.05.

    ## 

    ## 
    ## Null Hypothesis Test Result:
    ## The null hypothesis test was non-significant, t(78) = 1.297, p = 0.199, given an alpha of 0.05.

    ## 

    ## 
    ## Based on the equivalence test and the null-hypothesis test combined, we can conclude that the observed effect is statistically not different from zero and statistically equivalent to zero.

    ## 

![](README_files/figure-gfm/tost-ex-1.png)<!-- -->

If we read the output we can see that the `TOSTtwo.raw` function did
several things. First, it conducted the TOST using the values we
calculated from the simulated data. Second, it calculated an independent
samples t-test (like the one we did above). Finally, it created a plot
of the results and interpreted them in prose. We see that the
equivalence test is significant and the t-test is not. Together we use
this information to conclude that the group differences is not different
from 0 and statistically equivalent to 0. In other words, we find
evidence supported the notion that the L2 learners have acquired a
phonetic category for Spanish /t/ that is not different from that of the
native speakers.

The procedure is quite simple, though, in my case, it took awhile to
wrap my head around it. I think that the final plot is particulatly
illustrative, but it does require a bit of experience for it to be
useful. It is also true that if one does not have much experience
working with effect sizes, the intuition behind how these procedures
work and what they mean is not intuitive. I strongly suggest playing
with the web app [here](https://rpsychologist.com/d3/equivalence/) to
help build intuition for what each component of the process does and
together they can affect the outcome of the test.

# Refs

  - <http://daniellakens.blogspot.com/2016/12/tost-equivalence-testing-r-package.html>
  - <https://rpsychologist.com/d3/equivalence/>
  - <https://journals.sagepub.com/doi/full/10.1177/1948550617697177>
  - <https://cran.rstudio.com/web/packages/TOSTER/vignettes/IntroductionToTOSTER.html>
  - <https://github.com/Lakens/TOSTER>
  - <https://psyarxiv.com/v3zkt/>
