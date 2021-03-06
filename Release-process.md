# Semantic versioning (sic)

In overall, the regular expression pattern will be something like `\d+\.\d+(\.\d+|\-rc\d+)`.

Specifically:
  - x.y: major release, usually requires reindex from scratch
  - x.y.z: minor release
  - x.y-rcZ: release candidate, may require reindex from scratch

# Release criteria

Ideally, the following minimum criteria should be fulfilled before a new
final (i.e. non-prerelease) version is released:

  - The overall code coverage must be at least 70%
  - Sonarcloud reported bugs should not have any critical issues
  - No stoppers (meaning both Issues and Pull requests)
  - All bugs and enhancements must be evaluated (for final release also go through issues with given milestone and either fix them or retarget to next release by setting the new milestone)

# Checklist for releasing OpenGrok:

The below steps are common for both pre-release and final release:

1. check that latest build passes tests, test code coverage is above given threshold

   If extra care is needed (final release, significant changes since the last release, etc.), consider checking the following:
   - index fairly large code base, ideally multiple projects
   - deploy webapp
   - check UI:
     - history view
       - try comparing 2 distant revisions
       - check pagination
     - annotate view
     - directory listing
       - check sorting using multiple criteria
     - perform search using multiple fields across multiple projects

   The release is OK, once above is fulfilled to our satisfaction.

1. Trigger release creation

   `./dev/release INSERT_VERSION_HERE`
 
1. Wait for the build to finish and release created.

   Go to https://github.com/OpenGrok/OpenGrok/releases and edit the text
   of the release, e.g. adding list of issues fixed, whether complete reindex
   is necessary etc.

1. Check the https://github.com/OpenGrok/docker/blob/master/Dockerfile to see what version matching is done on the release string and if needed adjust accordingly to the latest stable release

1. Send announcement to opengrok-users@yahoogroups.com
   (the #opengrok Slack channel should receive the release info automatically thanks to Github hooks)

   If this is final release, update https://en.wikipedia.org/wiki/OpenGrok