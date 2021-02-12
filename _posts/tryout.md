One of my research interests are economic sanctions, and another one is
European Union decision-making. I found a very interesting article by
@biersteker2015 that combines these two topics. They categorize EU
sanctions into 3 categories: first those that are imposed without UNSC
approval, second those that are imposed “by order” of the UNSC and third
those that are imposed due to UNSC resolutions, but which exceed the
demanded intensity of the UNSC. These could, for example be cases where
the UN expressly states that member states should “exercise vigilance”
in their sanction regime.  
Really interesting - but now I want to find out more. How often did the
EU impose measures in each category? Has European decision-making
changed over time - and did the objectives of the EU differ depending on
the category?  
To find out more I will do some exploratory research using the
EUSANCT-Dataset by @weber2020a. This dataset ranges across the period
1990-2015 and includes all sanctions in place during that time for the
EU, the UN and the United States.

Data wrangling
==============

I start by loading the case-level dataset. I filter for cases where the
EU imposed sanctions, 81 of which were in place in the considered
period. Now I start to differentiate by category: I will create a
categorical variable that is 1 for cases where the EU imposed without
the UN, 2 for cases where the sanctions were mandated by the UN and 3
for cases where the strength of EU sanctions increased above UN sanction
strength. This is where it gets tricky: How to identify cases where the
sanctions by the EU exceeded the intensity of the sanctions of the UN?
Thankfully, the authors included a variable in their dataset that
measures the most intense sanction measure applied in a case, based on
the associated costs to the target regime @weber2020:9. It ranges from 1
- Visa bans across 2 targeted financial sanctions, 3 arms embargoes, 4
aid sanctions, 5 trade sanctions all the way to 6, economic embargo. I
will put a case into category 3 if the most intense measures by the EU
exceeded the most intense measures of the UN.  
Great, let’s look at the results!

    rm(list = ls())
    setwd("C:/Users/thies/OneDrive/00_Promotion/blog")
    library(dplyr)

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    library(readxl)
    library(ggplot2)
    library(knitr)
    EUSANCT <- read_excel("EUSANCT_Dataset_Case-level.xls")

    EUSANCT <- EUSANCT %>%
      filter(impositionEU == 1)%>%
      mutate(category = as.factor(if_else(impositionUN == 0, 1, 
                                if_else(impositionEU_target <= impositionUN_target, 2, 3))))

Variable distribution
=====================

    ggplot(EUSANCT, aes(x = category))+
      geom_bar()+
      labs(title ="Type of UN involvement in EU sanctions")

![](tryout_files/figure-markdown_strict/summary%20statistics-1.png) In
this first plot, I am interested in the number of cases per category.
Interesting, so actually the majority of cases were imposed without UN
involvement. The EU imposed cases more than 50 times without UNSC
mandate, while she exceeded the UNSC mandate in ~8 cases. She
implemented the UNSC directions ~18 times as instructed. I am wondering
whether there was a trend across time in the EU decisions. Let’s look at
that next!

Sanctions per category across time
==================================

In the next graph, I plotted the newly imposed sanctions per year over
time. The color indicates the category of UN involvement to which a
sanction case belongs.

    ggplot(EUSANCT, aes(x = impositionEU_year, y = impositionEU, fill =category))+
      geom_col(position = "stack")+
      labs(title = "Imposition of EU sanctions per year by type of UN involvement")+
      xlab("time")+
      ylab("Imposition of EU sanctions per year")

![](tryout_files/figure-markdown_strict/summary%20statistics%202-1.png)

Looking at the graph, it seems as though the number of UN-mandated
sanctions as a share of EU sanctions has decreased over time. Has the EU
grown more autonomous in its sanction decisions? Based on the graph, I
will determine 2000 as my cutoff year and see whether the share of
autonomous EU sanctions has increased after 2000 compared to before.

    EUSANCT <- EUSANCT %>%
      mutate(after2000 = if_else(impositionEU_year > 2000, 1, 0))

    t.test(EUSANCT$impositionUN ~ EUSANCT$after2000)

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  EUSANCT$impositionUN by EUSANCT$after2000
    ## t = 1.683, df = 78.992, p-value = 0.09633
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -0.03108588  0.37138072
    ## sample estimates:
    ## mean in group 0 mean in group 1 
    ##       0.3863636       0.2162162

From the output, we can see that the mean share of UN-mandated sanctions
before 2000 (in group 0) was indeed higher (~38.64%) than in the period
from 2001 to 2015 (~21.62%). Based on the conducted t-test, this
difference in means is statistically significant at a 90% significance
level. This means that with some certainty we can indeed say that the
European Union decision making regarding sanctions has grown more
autonomous. This development makes sense, given that we see more and
more efforts by the EU to establish itself as a global player, such as
the 2018 chemical weapons sanctions regime, the 2019 cyber attacks
sanctions regime and, most notably, the 2020 human rights violations and
abuses sanctions regime.

Sanction category by intended outcome
=====================================

Another question that I raised in the introduction is whether the
objectives of the EU differ depending on the UN involvement. The
motivation for this question is simple: The UN imposes sanctions due to
global threats to international peace and security. The EU on the other
hand imposes sanctions to protect common values across its member states
- most notably, human rights and democracy. Let’s look at the number of
EU sanctions per main issue, across the categories of UN involvement.

The EUSANCT-dataset contains a variable that identifies the main issue
of the sanction episode. It includes several possibilities such as to
contain political influence, incite leadership change and improve human
rights. If they ever were the main issue for the imposition of EU
sanctions, you can find them on the x-axis below.

    ggplot(EUSANCT, aes(as.factor(issue1), fill = category))+
      geom_bar()+
      theme(axis.text.x = element_text(angle = 90))+
      scale_x_discrete(name = "issue", 
                       breaks=c(1,2,3,4,5,6,7,8,9,10,11,12,13,14),
                       labels=c("contain political\n influence", "contain military\n behaviour", "leadership change", "release citizens, property, or material", "territorial dispute", "strategic materials", "alliance or alignment choice", "human rights", "weapons/materials proliferation", "support of\n non-state actors", "drug trafficking/\n
                                corruption", "violation of constitutional order", "fraud elections", "other"))

![](tryout_files/figure-markdown_strict/summary%20statistics%204-1.png)

The plot nicely shows the differences across the main issue of concern
for the imposition of sanctions for the EU and the UN. The EU imposed
most of its sanctions to induce leadership change and to improve the
human rights situation in the targeted country. The UN mostly acts to
contain military behaviour - as this is the mandate of the UN Security
Council.
