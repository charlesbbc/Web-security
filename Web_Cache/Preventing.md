# Preventing web cache deception vulnerabilities

* Use `Cache-Control` to mark dynamic resources, no-store and private.
* Configure CDN settings, so cache rules don't override cache-control header.
* Activate CDN protection. Can set cache rules verifying Content-Type matches the URL file extension. Ex: Cloudfare's Cache Deception Armor
* Verify discrepacies between origin and cache interpreting URL paths.
