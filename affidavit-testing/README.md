# Affidavit Testing

Some things are awkward or expensive to test: Is a game fun to play, was a
3rd-party asset created, did the user understand the new interface? Whenever
possible, we should take pains to break these things into smaller pieces that
*can* be automatically tested. When not possible, we can use **Affidavit
Testing**.

In this case, an [affidavit](https://en.wikipedia.org/wiki/Affidavit) means a
trusted document stating something along the lines of "*I solemnly swear that no
code has been modified since the tests were last run*". In the absence of
automatic testing, such an affidavit can give us assurances that we may
reasonably expect the code to function as intended without needing to repeat any
manual testing.

As an example, it may be desireable to verify that an application causes a
side-effect on the real world. Such side effects could be expensive,
destructive, or require manual cleanup. If the application logic involved in
these side effects can be relegated to a few files, then an affidavit can be
issued declaring that those file -- at the time of affidavit issuance -- can be
assumed to have been tested thoroughly by the person issuing the affidavit.
Automatic test suites can then ignore this section of the codebase if the
timestamp on the affidavit is *later than* the timestamp of each file that it
covers.

## An Example Implementation

Below is a `Makefile` for a project whose logic is split into two modules:
`main` and `external`. The unit tests account for everything in `main`, and
we strive to put as much logic in this module as possible. The logic that
antagonizes our sense of automatic testing we place in `external`, and we
strive to minimize the amount of logic in that module.

```make
SHELL=bash
.PHONY = test sign_affidavit;

test: _run_unit_tests affidavit.md;

_run_unit_tests:
	pytest tests/unit/*.py --cov=main

affidavit.md: external/spend_money.py external/cause_damage.py
	$(error "External logic has changed and must be validated manually")

sign_affidavit:
	@export OATH="I solemnly swear that I ran all the tests." && \
		read -p "$$OATH (Sign here): " SIGNATURE && \
		echo "$$OATH -$$SIGNATURE, $(shell date)" > affidavit.md
```

Per the usual logic of Makefiles, the logic to rebuild a target will only be
executed if that target's *dependencies* have been modified more recently than
the target itself. In this case, we deploy an unusual trick: the logic for
rebuilding `affidavit.md` *will always fail*. This is because we do not want the
affidavit to be recreated by any automatic process.

When a human or other sentient lifeform has determined that the `external` code
is fit for release, they may update the affidavit as follows:

```bash
make sign_affidavit
```

They will then be prompted to attest, on their honor as a software developer,
that the expected properties of the `external` module have not been violated by
any new changes.
