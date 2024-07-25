## Short Term

NOTE! pfsense remote logging currently DISABLED!

- ~~install pfelk via container and connect it to pfsense~~

- continue packaging elasticsearch 8.14.1
Next step is to figure out how to build the package to test it out. also see the notes in personal wiki for some stuff aa gave me.

## Long Term

### 0. baseline working example

- ~~install on ghost using docker~~
- ~~configure~~

#### Refs

- https://github.com/EmergentMind/pfelk/blob/main/install/docker.md
- [install notes](/docs/baseline_install_notes.md)

### 1. dependencies available in nixpkgs

- ~~add github release and security alert watches for:~~
    ~~elastic/kibana~~
    ~~elastic/logstash~~
    ~~elastic/elasticsearch~~
    ~~elastic/beats (not required for pfelk but is considered part of the elk stack?)~~

- pkg for elasticsearch 8.14.1

  - ~~update all-packages entries~~
  - ~~update nixpkgs/pkgs/search/elasticsearch/8.x.nix~~
  - confirm it builds
  - commit
  - update plugins and add hashes ../servers/search/elasticsearch/plugins.nix
  - confirm
  - verify FIXMES
  - need patch for 7.x like 7 does for 6.x?
  - update tests

  refer to this commit for some reference https://github.com/NixOS/nixpkgs/commit/afb57ff041463ed5586b2d350afa4fedf96c85e1

- pkg for logstash 8.14.1

  - update all-packages entries

- pkg for kibana 8.14.1

  - update all-packages entries

- pkg for maxmind geoip 7.0.1+

### 2. pfelk is nixos compatible

- assess existing installation process
- modify pfelk code to function on nxios
- PR changes to pfelk or publish pfelk-nix fork

### 3. package pfelk for nixpkgs

- pkg pfelk
- flake?
