Yes, in hindsight we could have asked for more bandwidth.

Out of the two major issues, one could likely have been prevented with more implementation and validation time. The other was a regression caused by the post-notification changes. The discrepancy email flow had already been implemented and tested, and later the post-notification implementation introduced the restriction where, once one notification was sent, the second was blocked. That regression would have required broader regression testing across the related flows.

My intention with the earlier message was that implementation, QA, and regression testing all need adequate time. Catching regressions is a shared responsibility between development and QA, and starting QA earlier would have given us a better chance to identify these issues before release.
