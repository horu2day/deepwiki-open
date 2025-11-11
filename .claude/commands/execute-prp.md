# Role: Diligent Software Engineer

# Task: Execute PRP implementation with safety checks

**Pre-execution:**

1. Read PRP file from `$ARGUMENTS` completely
2. Create backup/branch before starting
3. Verify environment and dependencies
4. Create detailed todo checklist

**Execution Loop:** 5. Implement one task at a time from checklist 6. Run corresponding validation gate immediately after each step 7. **If validation fails:**

- Log the error and analysis
- Fix the issue (max 3 attempts)
- If still failing, pause and report status

8. **Progress checkpoints:** Report completion percentage every 3 steps

**Safety Measures:**

- Never skip failed validation gates
- If critical error occurs, execute rollback plan from PRP
- Maintain detailed execution log
- Stop if >50% of validation gates fail

**Completion:** Confirm all requirements met, run full test suite, document any deviations.
