# blackduck-upgrade-plan
Upgrading Black Duck in large complex enterprises can be complicated.  This document is intended to try to smooth the process a bit.

This document is provided as-is, without warranty or liability.

Example usage for beginners:
- either just use this web ui
- or, use command line with vi
     - git clone https://github.com/pjsnps/blackduck-upgrade-plan.git                                                                               
     - cd blackduck-upgrade-plan/
     -   git checkout pjsnps-patch-1 # keeping on a branch until this is proven worthy                                      
     -   vi SynopsysBlackDuckPublicUpgradePlan_DRAFT_20210928PJ.md 
     -   pandoc -f markdown -t html -o - -i SynopsysBlackDuckPublicUpgradePlan_DRAFT_20210928PJ.md | links -dump | less -inRF  # check it on command line
     -   git commit -a -m "short message"  # commit to your local store
     -   git push  # push your change up to github
     -   \# check it in your browser:  https://github.com/pjsnps/blackduck-upgrade-plan/blob/pjsnps-patch-1/SynopsysBlackDuckPublicUpgradePlan_DRAFT_20210928PJ.md
     -   \# next time you edit
     -   git pull  # get any changes others might have made
     -   vi SynopsysBlackDuckPublicUpgradePlan_DRAFT_20210928PJ.md                                                                 
     -   git push

 Markdown syntax:  https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax
