# ICR benchmark mirror — laravel/framework

Read-only mirror of [laravel/framework](https://github.com/laravel/framework) (branch `13.x`) at the commit recorded in
`.icr-upstream`. It exists for one thing: run the framework's own Linux test job on GitHub-hosted runners and on
[ICR](https://www.irvyne.eu/icr/) (Irvyne Consulting Runners), and publish what happened. Upstream's workflows are kept
under `.github/upstream-workflows/` and do not run here. Nothing else was changed; verify it yourself:

    git fetch https://github.com/laravel/framework <commit from .icr-upstream>
    git diff --stat <that commit> HEAD

**Method.** `.github/workflows/icr-bench.yml` is upstream's `linux_tests` job for one matrix cell (PHP 8.4, PHPUnit
12.5.8, prefer-stable) with its service containers (MySQL 9.7, Redis 7, Memcached 1.6, DynamoDB Local): PHP and its
extensions are installed by `shivammathur/setup-php` (the Redis extension is compiled), dependencies come from Composer
without a cache, then PHPUnit runs the suite. Two edits: `--fail-on-deprecation` is dropped, so a deprecation in a
dependency released after the pinned commit cannot abort a measurement; and the `imagick` extension is disabled
(`:imagick`) on every leg, because with it loaded the AVIF test errors on Ubuntu 26.04 (its ImageMagick has no AVIF
encoder) while it passes or skips on GitHub's 24.04 — both sides now skip the same 32 imagick-guarded tests (570
skipped instead of 539) and run the other 15 100.
The service images are upstream's tags pinned by digest, and every job uploads its evidence (JUnit report, resolved
Composer dependencies, PHP version and extensions, image digests) as a run artifact, so two legs are compared on what
they actually ran.
Note the operating systems differ: `ubuntu-latest` was Ubuntu 24.04 (kernel 6.17, Docker 28) at the time of the runs,
ICR runs Ubuntu 26.04 (kernel 7.0, Docker 29); both are printed by the "Runner facts" step. It runs on `ubuntu-latest` (GitHub-hosted: on a
public repository a 4 vCPU / 16 GB runner; a private repository gets 2 vCPU / 7 GB) and on ICR's `icr-2c`, `icr-4c` and
`icr-8c` (2 / 4 / 8 vCPU with 8 / 16 / 32 GiB, one fresh virtual machine per job, destroyed afterwards). Wall time and
per-step time come from GitHub's own run data, so every published number links to its run. Several runs per leg; medians
and worst cases are published, not best cases.

**Who can run it.** Only Irvyne: a manual dispatch by a writer, then an approval on the `icr-bench` environment. Pull
requests, forks and comments cannot execute anything in this repository, and the runners are visible to this repository
alone within the organisation's runner group.

Laravel is the work of Taylor Otwell and the Laravel contributors; see `LICENSE.md`.
