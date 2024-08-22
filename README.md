# Survival_analysis

# Load necessary libraries
library(tidyverse)
library(janitor)
library(survival)
library(survminer)
library(tidyquant)  # For theme_tq
library(patchwork)  # For combining plots

# Load and prepare the data
customer_churn_tbl <- read_csv("G:/Other computers/My laptop/Oubaid/Documents/France/Alternance/DSTI/Survival Analysis/customer_churn.csv") %>%
    clean_names() %>%
    mutate(churn = ifelse(churn == "Yes", 1, 0)) %>%
    mutate_if(is.character, as_factor)

# Nonparametric estimation of survival for one or more groups
sfit <- survfit(Surv(tenure, churn) ~ contract, data = customer_churn_tbl)

# Kaplan-Meier survival curves with tidyquant theme
g1 <- ggsurvplot(
    sfit,
    conf.int = TRUE,
    data = customer_churn_tbl,
    palette = c("#E69F00", "#56B4E9", "#009E73"),
    ggtheme = theme_tq(),  # Apply tidyquant theme
    title = "Customer Churn Survival Plot"
)
print(g1)

# Adding a risk table to the Kaplan-Meier plot
g2 <- ggsurvplot(
    sfit,
    conf.int = TRUE,
    data = customer_churn_tbl,
    risk.table = TRUE,    # Add risk table
    ggtheme = theme_tq(), # Apply tidyquant theme
    palette = c("#E69F00", "#56B4E9", "#009E73"),
    title = "Customer Churn Survival Plot with Risk Table"
)

# Customize the plot and risk table
g2_plot <- g2$plot +
    labs(title = "Customer Churn Survival Plot")

g2_table <- g2$table +
    theme_tq() +
    theme(panel.grid = element_blank())

# Combine plot and risk table
combined_plot <- g2_plot / g2_table + plot_layout(heights = c(2, 1))
print(combined_plot)

# Check levels of gender to adjust color palette
unique_gender_levels <- unique(customer_churn_tbl$gender)
num_gender_levels <- length(unique_gender_levels)
print(num_gender_levels)  # Print number of levels

# Kaplan-Meier survival estimate with faceting by gender
# Adjust the palette size based on the number of gender levels
palette_colors <- RColorBrewer::brewer.pal(n = num_gender_levels, name = "Set1")

g3 <- ggsurvplot_facet(
    sfit,
    conf.int = TRUE,
    data = customer_churn_tbl,
    facet.by = "gender",
    nrow = 1,  # Arrange facets in one row
    ggtheme = theme_tq(), # Apply tidyquant theme
    palette = palette_colors,
    title = "Survival Plot by Customer Gender"
)
print(g3)

# Semi-parametric Cox regression
cox_model <- coxph(Surv(tenure, churn) ~ contract + gender, data = customer_churn_tbl)

# Summary of the Cox model
summary(cox_model)

# Representation graph of the Cox model
# Plotting survival curves based on Cox model
g4 <- ggsurvplot(
    survfit(cox_model),
    conf.int = TRUE,
    data = customer_churn_tbl,
    ggtheme = theme_tq(), # Apply tidyquant theme
    palette = c("#E69F00", "#56B4E9"),
    title = "Survival Curves from Cox Model"
)
print(g4)
