# 6. Constraints

Q: Purpose?
A: Enforce data integrity at the database layer—prevent invalid states.

Q: PRIMARY KEY?
A: Unique + NOT NULL identifier (one per table; can be composite).

Q: FOREIGN KEY?
A: Ensures referenced parent row exists; can cascade or restrict actions.

Q: UNIQUE vs PRIMARY KEY?
A: UNIQUE allows NULL (engine rules) & multiple per table; PK disallows NULL & only one.

Q: CHECK?
A: Boolean expression each row must satisfy (e.g., salary >= 0).

Q: DEFAULT?
A: Auto-supplied value when column omitted.

Q: NOT NULL?
A: Disallows NULL—combine with constraints for stronger semantics.

Q: Common oversight?
A: Missing index on foreign key column causing slow referential checks.
