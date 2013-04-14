AppGen
==========
**AppGen** generates and serves big apps for New York City's Big Apps
competition.

## Architecture
AppGen is divided into four major parts, each of which is a submodule
of the present repository. They are listed below in the order by which
data pass through them.

    1. Storeroom
    2. Kitchen
    3. Menu
    4. Delivery

Each major part contains more git submodules. We divide these submodules
into *input* and *output* submodules. A major part edits the contents of
its output submodules. The following major part includes this output
submodule as an input submodule. Each major part can be checked out
independently of the present repository. The structure of the submodules
enforces the immutability of data.

The storeroom downloads data and saves it in the most raw possible form.
Downloader scripts are stored in the storeroom repository, and the actual
downloads are stored in subdirectories. Almost **all** external data should be
downloaded in the storeroom. The one exception is interactive data requests
inside of the delivery (web application), like calling the Facebook API.

The kitchen takes the storeroom (with its various data submodules) as input and
converts them into a form that the generated web applications can use. This
involves randomization. For example, geoJSON files and app names are produced
in the kitchen. These data are all stored in one output submodule, called
"comstables", in one JSON file per random seed.

The menu takes the comstables as input and displays a webpage wherein people
can browse apps, claim an app and submit an app to CollabFinder. It saves the
information that people enter (their names and the apps that they claimed) in
either a submodule or a database that the delivery can access. Either way, we
call this output bucket the "assignments". Like the comstables, the
assignments are saved as one JSON file per random seed.

The delivery takes both the comstables and the assignments as input. It serves
the various specified apps based on the comstables and the assignments. (It
uses placeholder a assignment for apps that have not yet been assigned.) It
randomizes things that don't depend on the data, such as the map's tile server
and the overall app's theme; these things depend only on the random seed, so
they don't need to look at the earlier data. It also contains a mechanism for
rendering the various apps as static files.
