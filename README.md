# envlock
Layered env tool atop Miniconda/conda-pack: No conda Python packages; only pip + provided wheels; PySMT resolves conflicts. Binaries via popular manager per toolchain. Env packs fully offline. Spec .tar.gz locks everything for exact reproduction.

## Initial prompt

let's design a layered env manage tool on top of miniconda and conda-pack with restrictions:

user MUST NOT use conda channels to install ANY python packages. only pip is allowed to use. user MAY install provided wheels.
user use PySMT to resolve dependency confilcts by default. use MAY NOT know about PySMT.
user gets a `requirements.freeze.txt` from output of `pip freeze` by default.
user MAY use conda channels to install binaries they need, including java, javac, rustc, nodejs, npm, ollama, ..... the tool MUST choose the MOST POPULAR package manager for each language/toolchain. the user CAN NOT change the tool's choice.
user MAY pack the full env and ship to another machine. the packed env MUST NOT depended on internet.
user MAY pack the env spec into a single .tar.gz file. it contains everything locked and user provided wheels/other binaries. the tool CAN use that spec to reproduce the env EXACTLY the same with the help of internet.
