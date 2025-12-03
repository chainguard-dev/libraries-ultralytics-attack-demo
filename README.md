# Chainguard Libraries Ultralytics Demo

## Avoid Consuming Malware in Package Ecosystems

This demo showcases how to detect and prevent distribution-level supply chain attacks by using Chainguard Libraries as the safe source for open source. It compares different variants of the `ultralytics` package to highlight risks flagged by [malcontent-action](https://github.com/chainguard-dev/malcontent-action) which checks git commits using [malcontent diff](https://github.com/chainguard-dev/malcontent?tab=readme-ov-file#diff) 

## Usage

Update the setup.yaml to switch between different sources for ultralytics. Once setup.yaml the build workflow will kick off which checks for malware.

### Simulate Ultralytics Attack by comparing PyPI releases v8.3.39 vs v8.3.40

> ⚠️ **Note:** The `v8.3.40` release was removed from PyPI. This demo uses a snapshot from the [malcontent-samples repository](https://github.com/chainguard-dev/malcontent-samples/tree/main/python/2024.ultralytics/v8.3.40).

To test the malicious sample, create a new branch, then update `.github/workflows/setup.yaml` to switch sources and make a PR:

````yaml
env:
  ULTRALYTICS_SOURCE: "PYPI"
````

![image](https://github.com/user-attachments/assets/141ea426-b85b-4f84-af5d-846dadbcff4d)


### Prevent Ultralytics attack by deploying the safe Chainguard Build of Ultralyitics v8.3.40

To test the safe Chainguard Library update `.github/workflows/setup.yaml`:

````yaml
env:
  ULTRALYTICS_SOURCE: "CHAINGUARD"
````

You should see no changes in risk detected:
![image](https://github.com/user-attachments/assets/14724483-8842-4ed6-a6b9-7c7d73c16d55)

## Ultralytics Supply Chain Attack Background Information

This code was injected into the PyPI release artifacts and was not present in the public GitHub repository.  
See the advisory details here:
- [PYSEC-2024-154](https://github.com/pypa/advisory-database/blob/main/vulns/ultralytics/PYSEC-2024-154.yaml#L12-L13)
- [Ultralytics GitHub Pull Request](https://github.com/ultralytics/ultralytics/pull/18020?ref=blog.gitguardian.com#issuecomment-2525180194)

#### Pull Request introduced the blobs (that never got merged and were later force deleted)
- [GitHub Issue Comment 1](https://github.com/ultralytics/ultralytics/issues/18027#issuecomment-2526084417)
- [GitHub Issue Comment 2](https://github.com/ultralytics/ultralytics/issues/18027#issuecomment-2520462686)

#### Triggering Commit for Malicious Release
The commit `Cb260c243ffa3e0cc84820095cd88be2f5db86ca` is the triggering commit for the first malicious release.  
It bumps the version to the first known malicious version (`v8.3.41`) and critically removes a `github.actor` check that limited who could do `publish.yml` triggers from the main branch.  
This commit is authored by the `@UltralyticsAssistant` which strongly suggests that the attacker is in full control of the `@UltralyticsAssistant` identity at this point.  
- [GitHub Commit](https://github.com/ultralytics/ultralytics/commit/cb260c243ffa3e0cc84820095cd88be2f5db86ca)
- [GitHub Action Run](https://github.com/ultralytics/ultralytics/actions/runs/12168072999/job/33938058724)

#### Sigstore Attestation attestations for malicious distributions of v8.3.41 on PyPI:
- ultralytics-8.3.41.tar.gz https://search.sigstore.dev/?logIndex=153415338
- ultralytics-8.3.41-py3-none-any.whl https://search.sigstore.dev/?logIndex=153415340
- The release `v8.3.41` was uploaded to PyPI with a Trusted Publisher and valid attestations for each distribution, matching the `ultralytics/ultralytics` Trusted Publisher identity.

#### Branch References and Removal of Malicious Payload Files
Each branch name referenced a file in a commit that supposedly contained a malicious payload used during the exploitation. However, both those files were removed from GitHub following the incident.

#### Attack Payload
As forecasted in Woodruff’s analysis, the actual attack payload is a copy of the proof of concept from Adnan Khan's post on GitHub Actions cache poisoning.  
It uses the same Python memory dump tool and HTTP data exfiltration channel. The exfiltration occurred to a temporary HTTP webhook:  
`hxxps://webhook.site/9212d4ee-df58-41db-886a-98d180a912e6` (which has been deleted since then).  
No other mention of this webhook was observed in our GitHub dataset.

#### Additional Resources:
- [Yossarian’s Blog on the Attack](https://blog.yossarian.net/2024/12/06/zizmor-ultralytics-injection?ref=blog.gitguardian.com)
- [GitHub Issue Comment](https://github.com/ultralytics/ultralytics/issues/18027#issuecomment-2520085978)
- [GitGuardian Blog on the Attack](https://blog.gitguardian.com/the-ultralytics-supply-chain-attack-connecting-the-dots-with-gitguardians-public-monitoring-data/)
- [Woodruff’s Analysis Gist](https://gist.github.com/woodruffw/7d6a07077842508b85008e0267f7f3bb)
