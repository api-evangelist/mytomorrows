---
name: Generate a Trial Search Report (TSR)
description: >-
  Use the myTomorrows Enterprise Search API to turn a patient's condition into a
  Trial Search Report — resolve the condition, scope the search with the intake
  questionnaire, check matching study counts, then generate the report.
api: openapi/mytomorrows-enterprise-search-openapi.json
method: generated
source: derived from OpenAPI operations (operationIds verified in the spec)
operations:
- autocomplete_conditions_endpoint_v01_llm_autocomplete_conditions_synonyms_post
- questionnaire_endpoint_v01_search_questionnaire_post
- count_wrapper_endpoint_v01_wrapper_study_search_counts_get
- generate_tsr_endpoint_v01_search_generate_tsr_post
- request_tsr_endpoint_v01_search_request_tsr_post
---

# Generate a Trial Search Report (TSR)

The myTomorrows **Enterprise Search API** (host `https://enterprise-search.mytomorrows.com`,
also fronted at `https://trialsearchai-beta.mytomorrows.com`) is a FastAPI service
that powers Trial Search AI. All requests and responses are `application/json`.
The OpenAPI declares **no authentication scheme** — treat this as a beta surface and
do not send credentials unless myTomorrows provisions them for you.

Operations are exposed under two prefixes: bare `/v01/...` and the gateway
`/es/api/v01/...`. Use whichever your integration is pointed at; the operationIds
below are the bare `/v01` set.

## Conventions to respect

- **Versioning:** the version (`v01`) is pinned in the URL path.
- **Request bodies:** most POST endpoints accept a loose `JsonRequest` object
  (untyped JSON passthrough), so send the fields the product expects for each call.
- **Errors:** validation failures return HTTP `422` with a FastAPI envelope
  `{ "detail": [ { "loc": [...], "msg": "...", "type": "..." } ] }`
  (see `errors/mytomorrows-problem-types.yml`). There is no `Idempotency-Key`
  support, so do not assume safe automatic retries on non-GET calls.

## Steps

1. **Resolve the condition.** Call `autocomplete_conditions_endpoint_v01_llm_autocomplete_conditions_synonyms_post`
   (`POST /v01/llm/autocomplete_conditions_synonyms`) to normalize the patient's
   free-text condition into canonical conditions and synonyms. Optionally also call
   `autocomplete_country_endpoint_v01_llm_autocomplete_country_post` to resolve the
   country of interest.
2. **Scope the search.** Call `questionnaire_endpoint_v01_search_questionnaire_post`
   (`POST /v01/search/questionnaire`) to retrieve the intake questionnaire that
   frames a trial search, and collect the answers.
3. **Check availability.** Call `count_wrapper_endpoint_v01_wrapper_study_search_counts_get`
   (`GET /v01/wrapper/study/search/counts`) to see how many studies/trials match the
   scoped query before generating a full report.
4. **Generate the report.** Call `generate_tsr_endpoint_v01_search_generate_tsr_post`
   (`POST /v01/search/generate_tsr`) with the condition and questionnaire context to
   produce the Trial Search Report.
5. **(Optional) Request/deliver.** Call `request_tsr_endpoint_v01_search_request_tsr_post`
   (`POST /v01/search/request_tsr`) to register a report request for a case; delivery
   helpers such as `emailTSR_endpoint_v01_llm_email_tsr_post` can email the result.

## Notes

- This is clinical, regulated data (SOC 2 / ISO 27001 / HIPAA / GDPR per
  `https://trust.mytomorrows.com/`). Handle patient inputs and reports accordingly.
- The API is beta (`info.version` 0.1.0); operation shapes may change without a
  published deprecation policy.
