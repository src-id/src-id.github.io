# KSU Lidia sidecar verification pattern

Session pattern from KSU Lidia work:

- Claude Code ran in tmux session `ksu-lidia` at `/home/wanforge/www/_next-projects/ksu-lidia`.
- User wanted command control separate from Claude TUI, so a sidecar shell was created:
  ```bash
  tmux new-session -d -c /home/wanforge/www/_next-projects/ksu-lidia -s ksu-lidia-terminal bash
  ```
- Verification command used outside Claude TUI:
  ```bash
  npm --prefix apps/ksulidia run format:check && \
  npm --prefix apps/ksulidia run lint && \
  npm --prefix apps/ksulidia run type:check && \
  npm --prefix apps/ksulidia run build
  ```
- First run failed at Prettier check. Fix was `npm --prefix apps/ksulidia run format`.
- Second run failed at TypeScript because app `tsconfig.json` included test files that required absent `vitest` and `@testing-library/react` deps. Minimal app-build fix excluded tests:
  ```json
  "exclude": ["node_modules", "tests", "src/**/__tests__"]
  ```
- Final full chain exited 0: format check pass, lint pass, typecheck pass, Next build pass.
- Local reference dump `Tukang Digital - Koperasi/` was ignored with an anchored `.gitignore` rule:
  ```gitignore
  /Tukang Digital - Koperasi/
  ```

Reusable lesson: keep interactive agent session alive for reasoning; use sidecar shell for deterministic verification and git hygiene.
