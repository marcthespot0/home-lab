# AGENTS
1. Repo stores Argo CD-delivered Kubernetes manifests; keep files self-contained per app.
2. Use two-space YAML indenting, '---' between documents, and no tabs.
3. Order resources Namespace -> CRDs -> RBAC -> Services -> Workloads -> add-ons.
4. Pin exact apiVersion/kind pairs; update cluster CRDs together.
5. Always set metadata.name/namespace explicitly; names stay kebab-case and namespace equals app.
6. Provide labels app.kubernetes.io/name|component consistently on every resource and selector.
7. Keep annotations alphabetized; add argocd sync options only when upstream requires.
8. Keep string quoting consistent (double quotes for templated values, bare scalars otherwise).
9. Secrets/configs should reference ExternalSecret or ConfigMap data; never embed passwords inline.
10. Validate syntax with `yamllint *.yaml`; single file: `yamllint homepage.yaml`.
11. Schema-test manifests via `kubeconform -strict -summary *.yaml`; single file: `kubeconform homepage.yaml`.
12. Run safety diff before deploy using `kubectl diff -f <file>` or `kubectl diff -f .`.
13. Dry-run/apply with `kubectl apply --server-dry-run=server -f <path>`; drop the flag only when coordinated since Argo CD owns production.
14. Keep ConfigMaps/Secrets data keys upper snake case and mirror the consuming env var names.
15. When referencing ServiceAccounts, ensure Secrets exist beforehand (see homepage.yaml example).
16. Avoid YAML anchors/aliases unless eliminating large duplication; clarity beats brevity.
17. Comment only to explain non-obvious platform constraints or vendor requirements.
18. Introduce new CRDs before their instances in the same file to avoid Argo sync failures.
19. There are no Cursor or Copilot rule files; treat this guide as the source of truth.
