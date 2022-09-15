# R & Python Import Extensions for Inkscape

Inkscape extensions for executing R/Py scripts from Inkscape to represent the resulting R/Py plot inside the Inkscape canvas.

# Requirements

R or Python and Inkscape should be installed on the platform. On Windows Inkscape come with own Python installation but installation of new packages there is restricted. It is very likely you have your own executable which need to specified in extension file `py_import_windows.py`.
I'm using `reticulate` 

# Extension set up

1. Make sure the `PATH` to` Rscript` is set in the Environment Variables of your system.

2. Files from `extensions` folder of this repository:

- `r_import.inx`

- `r_import.py` 

- `py_import.inx`

- `py_import_linux.py` or `py_import_windows.py`

should be copied to the User Extensions directory which is listed at `Edit`>`Preferences`>`System` - `User Extensions:` in Inkscape.

3. Make sure R or Python packages from examples are installed 

# R scripts

In order for the script to run correctly, it must meet the following convention:

```
#!/usr/bin/env Rscript

<-- Your code goes here -->

ggsave(filename = commandArgs(trailingOnly = TRUE)[1])

```

# Python scripts

In order for the script to run correctly, it must meet the following convention:

```
#!/usr/bin/env python

<-- Your code goes here -->

fig.savefig(sys.argv[1], format='svg', dpi=1200)

```

# R scripts containing Python scipts

In order for the R script to run correctly, it must meet the following convention:

```
#!/usr/bin/env Rscript
file = commandArgs(trailingOnly = TRUE)[1]
library(reticulate)
Sys.setenv(RETICULATE_MINICONDA_PATH = 'C:/Users/Public/r-miniconda')
reticulate::repl_python()

<-- Your Python code goes here -->

ax.set(aspect=1)
# your python code ends here
fig.savefig(r.file, format='svg', dpi=1200)
exit

```

# Why does it work?

After importing (or opening) R script with Inkscape: 

![](images/Capture-open.PNG)

Inkscape builds command like:

`Rscript script.R output.svg` which is executed by the system. At that time your script creates `args` variable where keeps the name of the output. That name is next passed to `ggsave` and your `plot` is saved there. At the end Inkscape loads the output into canvas. Easy !

When using `Import` a popup will show:

![](images/Capture-import.PNG)

The same logic applies to running Python scripts.

# Examples

## Iris

In the `examples` folder you can find `iris.R` script with following content: 

```
#!/usr/bin/env Rscript
# Your code starts here
library(tidyverse)

iris %>%
  ggplot() +
  aes(x = Petal.Length,
      y = Petal.Width,
      colour = Species) +
  geom_point()

# Your code ends here
ggsave(filename = commandArgs(trailingOnly = TRUE)[1])

```

after import you should see in Inkscape:

![](images/Capture-iris.PNG)

## Rose

Another example, script `rose.R`:

```
#!/usr/bin/env Rscript
# Your code starts here
library(tidyverse)
library(sf)

st_rose = function(x) {
  p = x %>% select(p) %>% pull()
  q = x %>% select(q) %>% pull()
  n = x %>% select(n) %>% pull()
  tibble(theta = seq(0, n * pi, length.out = 100)) %>%
    mutate(r = 1 * cos(p / q * theta)) %>%
    mutate(x = r * cos(theta)) %>%
    mutate(y = r * sin(theta)) %>%
    select(x, y) %>%
    as.matrix() %>%
    list() %>%
    st_multilinestring() %>%
    st_sfc()}

tibble(p = 3, q = 5) %>% 
  mutate(n = ifelse((p * q) %% 2 == 0, 2 * q, 1 * q)) %>%
  st_rose() %>%
  ggplot() +
  geom_sf()

# Your code ends here
ggsave(filename = commandArgs(trailingOnly = TRUE)[1])

```

after open you should see in Inkscape:

![](images/Capture-rose.PNG)

The rose curve is well described at https://en.wikipedia.org/wiki/Rose_(mathematics). Using 'SIMPLE FEATURES' to build them is a bit extravagant. Truth :)

## Tulip

The last example from script `tulip.R` :

```
#!/usr/bin/env Rscript
args = commandArgs(trailingOnly = TRUE)
# Your code starts here
library(tidyverse)
library(sf)

plot <-
  "https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs?request=GetFeature&service=WFS&version=2.0.0&typeName=cbs_gemeente_2022_gegeneraliseerd&outputFormat=json" %>%
  st_read() %>%
  ggplot() +
  geom_sf() +
  theme_void()

# Your code ends here
ggsave(filename = args[1] , plot = plot)

```

![](images/Capture-tulip.PNG)

## Flowers

This is rather advanced example, consider reading https://www.tidytextmining.com/ before giving up. Shortly, we read data from github, count words in flower names and plot most frequent 200 tokens so that more frequent tokens have bigger font size. Note that here I'm not specifying plot name, `ggsave` takes `last_plot()`.

```
#!/usr/bin/env Rscript
args = commandArgs(trailingOnly = TRUE)
# Your code starts here

if (!require("ggwordcloud"))
  install.packages("ggwordcloud")
if (!require("tidytext"))
  install.packages("tidytext")

library(tidyverse) # general meta package
library(ggwordcloud) # for world cloud
library(tidytext) # for NLP
"https://gist.githubusercontent.com/researchranks/ffe24c33df30e64f51271ddec83b4af6/raw/0e15dabe9b54611288cf92f93e1bfa288e150448/flower-and-plant-names.csv" %>%
  read_csv(col_names = FALSE) %>%
  mutate(linenumber = row_number()) %>%
  unnest_tokens(word, X1)  %>%
  count(word, sort = T) %>%
  top_n(200) %>%
  ggplot() +
  geom_text_wordcloud_area(aes(label = word, size = n)) +
  scale_size_area(max_size = 15)

# Your code ends here
ggsave(filename = args[1])

```
![](images/Capture-flowers.PNG)

## Snowflake

Example from script `koch.py` showing `matplotlib` in action.


```

#!/usr/bin/env python
import sys
# start of your script

import numpy as np
import matplotlib.pyplot as plt


def koch_snowflake(order, scale=10):
    """
    Return two lists x, y of point coordinates of the Koch snowflake.

    Parameters
    ----------
    order : int
        The recursion depth.
    scale : float
        The extent of the snowflake (edge length of the base triangle).
    """
    def _koch_snowflake_complex(order):
        if order == 0:
            # initial triangle
            angles = np.array([0, 120, 240]) + 90
            return scale / np.sqrt(3) * np.exp(np.deg2rad(angles) * 1j)
        else:
            ZR = 0.5 - 0.5j * np.sqrt(3) / 3

            p1 = _koch_snowflake_complex(order - 1)  # start points
            p2 = np.roll(p1, shift=-1)  # end points
            dp = p2 - p1  # connection vectors

            new_points = np.empty(len(p1) * 4, dtype=np.complex128)
            new_points[::4] = p1
            new_points[1::4] = p1 + dp / 3
            new_points[2::4] = p1 + dp * ZR
            new_points[3::4] = p1 + dp / 3 * 2
            return new_points

    points = _koch_snowflake_complex(order)
    x, y = points.real, points.imag
    return x, y

x, y = koch_snowflake(order=5)

fig = plt.figure(figsize=(8, 8))
plt.axis('equal')
plt.fill(x, y)


# end of your script
fig.savefig(sys.argv[1], format='svg', dpi=1200)

```

# Extra



# Notes

- `ggsave` saves a ggplot (or other grid object) with sensible defaults so that can be used to produce SVG

- method `plot()` in R doesn't work

- R working directory is extensions directory. To read data from file you need to specify full path or change working directory with `setwd()`

- `matplotlib.pyplot.savefig` saves the current figure.

- when using `reticulate` don't override `r` variable reserved for R environment

- when using `reticulate` you might need to specify `RETICULATE_MINICONDA_PATH` like `Sys.setenv(RETICULATE_MINICONDA_PATH = 'C:/Users/Public/r-miniconda')`

# References

https://inkscape.gitlab.io/extensions/documentation/tutorial/my-first-import-extension.html
