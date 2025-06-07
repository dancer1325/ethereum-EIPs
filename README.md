# Ethereum Improvement Proposals (EIPs)

* EIPs repository
  * | NOWADAYS,
    * -> [ERC repository](https://github.com/ethereum/ercs) + this repo 
      * Reason: 🧠 [here](https://github.com/ethereum/EIPs/pull/7206) 🧠
      * NEW ERCs & updates -> | [ERC repository](https://github.com/ethereum/ercs)
    * goal
      * 💡standardize Ethereum's documentation & conventions (past & ongoing)💡
        * == “source of truth”
        * == technical specifications

* [EIP-1](https://eips.ethereum.org/EIPS/eip-1)
  * how to publish
    * EIPs
    * ERCs

* [index.html](index.html)
  * hosted |[status page](https://eips.ethereum.org/)

* ERCs & EIPs' categories
  - [Core EIPs](core.html)
    - == Ethereum consensus protocol's improvements  
    - hosted [here](https://eips.ethereum.org/core)
  - [Networking EIPs]
    - hosted [here](https://eips.ethereum.org/networking)
    - == Ethereum's peer-to-peer networking layer  
  - [Interface EIPs](interface.html)
    - hosted [here](https://eips.ethereum.org/interface)
    - == Ethereum's standardize interfaces 
      - how users & applications -- interact with the -- blockchain
  - [ERCs](https://eips.ethereum.org/erc)
    - application layer standards /
      - specify how application1 running | Ethereum -- can interact with -- application2 running | Ethereum 
  - [Meta EIPs](https://eips.ethereum.org/meta
    - == miscellaneous improvements
  - [Informational EIPs](https://eips.ethereum.org/informational)
    - == NON-standard improvements / NOT require consensus

* cycle
  * raise up an idea | [Ethereum Magicians](https://ethereum-magicians.org/) or [Ethereum Research](https://ethresear.ch/t/read-this-before-posting/8)
  * if consensus is reached -> follow [EIP-1](https://eips.ethereum.org/EIPS/eip-1) to write one

* if you have questions about how to implement it -> [Ethereum Stack Exchange](https://ethereum.stackexchange.com)

* if you want to become an EIP Editor -> read [EIP-5069](https://eips.ethereum.org/EIPS/eip-5069)

## Preferred Citation Format

* ALL ERC completed -> hosted | https://eips.ethereum.org/
  * _Example:_ [EIP-1](https://eips.ethereum.org/EIPS/eip-1)

* ALL ERCs NOT hosted | https://eips.ethereum.org/ == working paper ("draft", "review", or "last call")

## Validation and Automerging

* TODO:All pull requests in this repository must pass automated checks before they can be automatically merged:

- [eip-review-bot](https://github.com/ethereum/eip-review-bot/) determines when PRs can be automatically merged [^1]
- EIP-1 rules are enforced using [`eipw`](https://github.com/ethereum/eipw)[^2]
- HTML formatting and broken links are enforced using [HTMLProofer](https://github.com/gjtorikian/html-proofer)[^2]
- Spelling is enforced with [CodeSpell](https://github.com/codespell-project/codespell)[^2]
  - False positives sometimes occur
* When this happens, please submit a PR editing [.codespell-whitelist](https://github.com/ethereum/EIPs/blob/master/config/.codespell-whitelist) and **ONLY** .codespell-whitelist
- Markdown best practices are checked using [markdownlint](https://github.com/DavidAnson/markdownlint)[^2]

[^1]: https://github.com/ethereum/EIPs/blob/master/.github/workflows/auto-review-bot.yml
[^2]: https://github.com/ethereum/EIPs/blob/master/.github/workflows/ci.yml

It is possible to run the EIP validator locally:

Make sure to add cargo's `bin` directory to your environment (typically `$HOME/.cargo/bin` in your `PATH` environment variable)

```sh
cargo install eipw
eipw --config ./config/eipw.toml <INPUT FILE / DIRECTORY>
```

## Build the status page locally

### Install prerequisites

1. Open Terminal.

2. Check whether you have Ruby 3.1.4 installed
* Later [versions are not supported](https://stackoverflow.com/questions/14351272/undefined-method-exists-for-fileclass-nomethoderror).

   ```sh
   ruby --version
   ```

3. If you don't have Ruby installed, install Ruby 3.1.4.

4. Install Bundler:

   ```sh
   gem install bundler
   ```

5. Install dependencies:

   ```sh
   bundle install
   ```

### Build your local Jekyll site

1. Bundle assets and start the server:

   ```sh
   bundle exec jekyll serve
   ```

2. Preview your local Jekyll site in your web browser at `http://localhost:4000`.

More information on Jekyll and GitHub Pages [here](https://docs.github.com/en/enterprise/2.14/user/articles/setting-up-your-github-pages-site-locally-with-jekyll).
