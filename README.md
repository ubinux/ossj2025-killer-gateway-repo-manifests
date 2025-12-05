# ossj2025-killer-gateway-repo-manifests

## Howto use

### Prerequisites

* Install all packages required to build Yocto 5.1.
* Install git-repo.
* Install python3-requests.

### Release Products (3/28/2025)

#### Build & Dploy step

```
cd ${WORKDIR}

MANIFEST_REPO="https://github.com/ubinux/ossj2025-killer-gateway-repo-manifests.git"
TAG="refs/tags/mar-28"

repo init \
  -u ${MANIFEST_REPO} \
  -b "${TAG}" \
  -m default.xml \
  --repo-url=https://github.com/GerritCodeReview/git-repo \
  --repo-rev=stable \
  --no-clone-bundle
repo sync

TEMPLATECONF=../meta-killer-gateway/conf/templates/default source poky/oe-init-build-env build

bitbake killer-gateway-image
```

#### Trace Vulnerability step (yocto cve-check)

##### yocto bitbake cve-check

```
cd ${WORKDIR}
TEMPLATECONF=../meta-killer-gateway/conf/templates/default source poky/oe-init-build-env build

echo "INHERIT+=\"cve-check\"" >> conf/local.conf
bitbake killer-gateway-image
```

##### Potentially AE Vul check

```
cd ${WORKDIR}
git clone https://github.com/ubinux/ossj2025-cve-sandbox.git

# Collect CVEs of killer-gateway
python3 ./ossj2025-cve-sandbox/script/check-cve-for-images-json.py build/tmp/deploy/images/*/*.rootfs.json > unpatch.txt

# CISA KEV Actively Exploited matching
wget -O known_exploited_vulnerabilities.json https://raw.githubusercontent.com/cisagov/kev-data/develop/known_exploited_vulnerabilities.json
python3 ./ossj2025-cve-sandbox/script/match_cves_vs_kev.py -i unpatch.txt -j known_exploited_vulnerabilities.json --out-exploited actively_exploited_kev.csv --out-nonexp non_actively_exploited_kev.csv
echo "Actively Exploited list(CISA KEV)"
cat actively_exploited_kev.csv

# ENISA Actively Exploited matching
python3 ./ossj2025-cve-sandbox/script/fetch_euvd_exploited_all_json.py --out euvd_exploited_all.json
python3 ./ossj2025-cve-sandbox/script/match_cves_vs_euvd.py -i unpatch.txt -j euvd_exploited_all.json --out-exploited actively_exploited_euvd.csv --out-nonexp non_actively_exploited_euvd.csv
echo "Actively Exploited list(ENISA API)"
cat actively_exploited_kev.csv
```
