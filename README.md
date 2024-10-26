<p align="center">
    <img width="180px" src="man/figures/parsim-logo.png" alt="parsim logo"/>
</p>

<h1 align="center">
    Parallel Simulator
</h1>

<p align="center">
    <a href="https://www.r-pkg.org/pkg/parSim"><img src="https://www.r-pkg.org/badges/version/parSim" alt="CRAN version"/></a>
    <a href="https://cran.r-project.org/web/checks/check_results_parSim.html"><img src="https://badges.cranchecks.info/worst/parSim.svg" alt="CRAN checks"/></a>
    <a href="https://github.com/SachaEpskamp/parSim/actions"><img src="https://github.com/SachaEpskamp/parSim/workflows/R-CMD-check/badge.svg" alt="R-CMD-check" /></a>
</p>

## Description.

`parSim` is an `R` package that provides convenient functionality to perform
flexible simulations in parallel using
[`parabar`](https://parabar.mihaiconstantin.com) backends.

## Installation

- to install from CRAN run `install.packages("parSim")`
- to install the latest version from GitHub run `remotes::install_github("SachaEpskamp/parSim")`

## Example

```r
# Load the package.
library(parSim)

# Determine a function to evaluate for each simulation condition.
bias <- function(x, y) {
    # Perform some computation.
    result <- abs(x - y)

    # Return the result.
    return(result)
}

# Run the simulation.
results <- parSim(
    # The simulation conditions.
    sample_size = c(50, 100, 250),
    beta = c(0, 0.5, 1),
    sigma = c(0.25, 0.5, 1),

    # The expression to evaluate for each simulation condition.
    expression = {
        # Generate the data.
        x <- rnorm(sample_size)
        y <- beta * x + rnorm(sample_size, sigma)

        # Fit the model.
        fit <- lm(y ~ x)

        # Compute the relevant quantities.
        beta_estimate <- coef(fit)[2]
        r_squared <- summary(fit)$r.squared
        bias <- bias(beta, beta_estimate)

        # Return in a compatible format.
        list(
            beta_estimate = beta_estimate,
            r_squared = r_squared,
            bias = bias
        )
    },

    # The number of replications.
    replications = 100,

    # The conditions to exclude.
    exclude = sample_size == 50 | beta <= 0.5,

    # The variables to export.
    exports = c("bias"),

    # No packages are required for export.
    packages = NULL,

    # Do not save the results.
    save = FALSE,

    # Execute the simulation on a single core.
    cores = 1,

    # Show the progress bar.
    progress = TRUE
)

# Print the head of the results.
head(results)
```

We can also use the `configure_bar` function (i.e., exported for from the
[`parabar`](https://parabar.mihaiconstantin.com) package) to customize the
progress bar.

```r
# Configure the progress bar.
configure_bar(
    type = "modern",
    format = "[:bar] [:percent] [:elapsed]",
    show_after = 0.15
)
```

Then, we can proceed with running the simulation as before.

```r
# Run the simulation again with more cores and the updated progress bar.
results <- parSim(
    # The simulation conditions.
    sample_size = c(50, 100, 250),
    beta = c(0, 0.5, 1),
    sigma = c(0.25, 0.5, 1),

    # The expression to evaluate for each simulation condition.
    expression = {
        # Generate the data.
        x <- rnorm(sample_size)
        y <- beta * x + rnorm(sample_size, sigma)

        # Fit the model.
        fit <- lm(y ~ x)

        # Compute the relevant quantities.
        beta_estimate <- coef(fit)[2]
        r_squared <- summary(fit)$r.squared
        bias <- bias(beta, beta_estimate)

        # Return in a compatible format.
        list(
            beta_estimate = beta_estimate,
            r_squared = r_squared,
            bias = bias
        )
    },

    # The number of replications.
    replications = 1000,

    # The conditions to exclude.
    exclude = sample_size == 50,

    # The variables to export.
    exports = c("bias"),

    # No packages are required for export.
    packages = NULL,

    # Save the results to a temporary file.
    save = TRUE,

    # Execute the simulation in parallel.
    cores = 4,

    # Show the progress bar.
    progress = TRUE
)

# Print the tail of the results.
tail(results)
```

Finally, we can also plot the results, for example, using the `ggplot2` package
as follows.

```r
# Load relevant libraries.
library(ggplot2)
library(tidyr)

# Pre-process the results in long format for plotting.
results_long <- tidyr::gather(results, metric, value, beta_estimate:bias)

# Make factors with nice labels for plotting.
results_long$sigma_factor <- factor(
    x = results_long$sigma,
    levels = c(0.25, 0.5, 1),
    labels = c("Sigma: 0.025", "Sigma: 0.5", "Sigma: 1")
)

# Plot.
ggplot2::ggplot(
    results_long, ggplot2::aes(
        x = factor(sample_size), y = value, fill = factor(beta))
    ) +
    ggplot2::facet_grid(
        metric ~ sigma_factor, scales = "free"
    ) +
    ggplot2::geom_boxplot() +
    ggplot2::theme_bw() +
    ggplot2::xlab("Sample size") +
    ggplot2::ylab("") +
    ggplot2::scale_fill_discrete("Beta")
```
