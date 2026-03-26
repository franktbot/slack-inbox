# Build v{N} Ship Checklist

## Checklist Discipline
- Update checklist items to `[x]` immediately after completion.
- Keep only incomplete required items as `[ ]`.
- Add blocker notes for any unchecked required item.
- Build cannot close and `/recap` cannot run until required checklist items are all `[x]`.

## Pre-Deploy
- [ ] All tasks marked done in `TASKS.md`
- [ ] All guards pass
- [ ] Review status is `PASS` in `REVIEW.md`
- [ ] Code reviewed

## Deploy
- [ ] Changes deployed to target environment
- [ ] Configuration/environment variables updated if needed

## Post-Deploy Smoke
- [ ] Core functionality verified
- [ ] No regressions in existing flows

## Rollback
- [ ] Rollback procedure documented
- [ ] Rollback trigger conditions defined
