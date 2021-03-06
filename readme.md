AppGen
==========
**AppGen** generates and serves big apps for New York City's Big Apps
competition. This repository is the main repository for all of AppGen's
sub-repositories. Once we get tired of using Facebook for project management,
we might decide to use the present repository's issue tracker for the overall
project.

## How to

Clone like so.

    git clone --recursive git@github.com:appgen/main.git appgen

With nested submodules, you'll probably want to commit a few submodules after
you've made a bunch of changes.

    ./commit

## Architecture
AppGen is divided into three major parts, each of which is a submodule
of the present repository. They are listed below in the order by which
data pass through them.

1. Pantry
2. Kitchen
3. Menu

Each major part contains more git submodules. We divide these submodules
into *input* and *output* submodules. A major part edits the contents of
its output submodules. The following major part includes this output
submodule as an input submodule. Each major part can be checked out
independently of the present repository. The structure of the submodules
enforces the immutability of data.

### Components

#### Pantry
The pantry downloads data and saves it in the most raw possible form.
Downloader scripts are stored in the pantry repository, and the actual
downloads are stored in subdirectories. Almost **all** external data should be
downloaded in the pantry. One exception is interactive data requests
inside of the menu (web application), like calling the Facebook API. The
pantry reads some of the data it downloads, but only enough to download more
data.

#### Kitchen
The kitchen takes the pantry (with its various data submodules) as input and
converts them into a form that the generated web applications can use. This
involves randomization. For example, geoJSON files and app names are produced
in the kitchen. These data are all stored in one output submodule, called
"comestibles", in one JSON file per random seed.

#### Menu
The menu takes the comestibles as input and displays a webpage wherein people
can browse apps, claim an app, deploy the app to Heroku and submit the app to
CollabFinder. It saves the information that people enter (their names and the
apps that they claimed) in a database of sorts.

It also serves the various specified apps based on the comestibles and the
claims. (It uses placeholder for apps that have not yet been claimed.)

It randomizes things that don't depend on the data, such as the map's tile
server and the overall app's theme; these things depend only on the random
seed, so they don't need to look at the earlier data.

It queries some external services. These include

* Facebook (for authentication)
* Url2PNG (for taking screenshots of generated apps)

It also contains a mechanism for rendering the various apps as static files.
These static files are saved in `deliveries`.

I (Tom) had considered separating the menu into separate apps for the generated
apps and the apps browser, but there seemed to be enough two-way communication
between these two components that they should be seen as one.

### Submodule conventions
All git repositories are stored on GitHub under the appgen organization.
Output submodules are specified (in `.gitmodules`) with the standard SSH
protocal. Input submodules are specified with git-ssh (`git://`).

When pushed to GitHub, repositories used for passing data between components 
should be prefixed with the string `cache-`. For example, a `socrata` submodule
inside of the `pantry`, with the path "`pantry/socrata`" in the present
repository, is named [pantry-socrata](https://github.com:appgen/cache-socrata).

The present repository is named [main](https://github.com:appgen/main).
Other repositories belonging to the appgen organization but not inside of the
present super-repository should use names that begin with a period (`.`).
For example, if we conduct an analysis of our server logs, we might name it
"`.server-log-analysis`".

### Software that each component uses
The pantry contains many tiny programs for downloading data. They are quite
independent, and each can be written in a different language.

The various datasets are connected in the kitchen, so it is more important that
they be able to interface nicely and that software be written in a way that
makes debugging easy. Also, much computation occurs in the kitchen, so the
language must be reasonably fast. Finally, some natural language processing is
going to happen. Considering all of these things, the pantry will probably
be written in Python, with NLTK, pandas, numpy, scikit-learn, &c. That said,
it wouldn't be hard to write some of this in other languages.

The menu is one Sinatra application. Let's keep all of this in Ruby.

## Other resources
Big Apps information

* http://www.nycedc.com/services/nyc-bigapps/past-competitions
* http://nycbigapps.com/prizes
* http://nycbigapps.com/rules
* http://2010.nycbigapps.com

Federal data is now allowed. It's also Socrata.

https://explore.data.gov/catalog/raw

