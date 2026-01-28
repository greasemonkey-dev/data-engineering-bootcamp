# Instructor Guide - Data Engineering Bootcamp Quizzes

## Quick Start

### For First-Time Use:
1. Review the 4 quiz files in their respective chapter folders
2. Read QUIZ_SUMMARY.md for overall assessment strategy
3. Use INDEX.md as the student-facing navigation guide
4. Consult QUESTION_ANALYSIS.md for detailed pedagogical notes

### For Canvas Integration:
1. Create 4 new quizzes in Canvas (one per chapter)
2. Copy questions from each quiz.md file
3. Input answer keys (see Answer Key Matrix below)
4. Set to allow multiple attempts for formative assessment
5. Enable answer explanations to show after submission

---

## Answer Key Matrix

### Chapter 1: Python & SQL Foundations
| Q# | Answer | Concept |
|----|--------|---------|
| 1 | B | Generators and Lazy Evaluation |
| 2 | B | Context Managers |
| 3 | B | Window Functions |
| 4 | A | Window Function Frames |
| 5 | B | Common Table Expressions |
| 6 | B | List Comprehensions vs Generators |
| 7 | B | Denormalization Trade-offs |
| 8 | A | SQL Aggregation Functions |
| 9 | B | Decorators |
| 10 | B | Query Optimization |

### Chapter 2: DevOps - Git & Docker
| Q# | Answer | Concept |
|----|--------|---------|
| 1 | B | Docker Containers |
| 2 | B | Dockerfile and Image Relationship |
| 3 | B | Git Merge vs Rebase |
| 4 | A | Git Rebase Best Practices |
| 5 | B | Docker Compose |
| 6 | B | Docker Layer Caching |
| 7 | B | Pull Request Workflow |
| 8 | C | Git Security Best Practices |
| 9 | B | Docker Image Optimization |
| 10 | B | Branching Strategies |

### Chapter 3: Database Design & Modeling
| Q# | Answer | Concept |
|----|--------|---------|
| 1 | B | Database Normalization |
| 2 | A | Second Normal Form |
| 3 | B | Database Indexes |
| 4 | B | Index Types and Selection |
| 5 | B | Primary and Foreign Keys |
| 6 | B | Schema Design |
| 7 | B | Denormalization in Data Warehousing |
| 8 | B | Index Design |
| 9 | A | Slowly Changing Dimensions |
| 10 | B | Query Execution Plan Analysis |

### Chapter 4: Data Warehousing & BigQuery
| Q# | Answer | Concept |
|----|--------|---------|
| 1 | B | Fact Tables |
| 2 | B | Star vs Snowflake Schema |
| 3 | B | BigQuery Partitioning |
| 4 | C | BigQuery Optimization |
| 5 | A | OLTP vs OLAP |
| 6 | B | Dimension Table Design |
| 7 | A | Denormalization Trade-offs |
| 8 | B | BigQuery Optimization |
| 9 | B | Slowly Changing Dimensions Type 2 |
| 10 | B | Schema Design and Cost Analysis |

---

## Assessment Strategies

### Strategy 1: Chapter Quizzes (Formative)
**When:** After each chapter (end of week)
**Format:** Individual self-assessment
**Configuration:**
- Allow unlimited attempts
- Show answer explanations after submission
- Do not include in final grade
- Use for identifying learning gaps

**Feedback Notes:**
- Share model answers with explanations
- Highlight common misconceptions
- Hold office hours for questions >30% missed

### Strategy 2: Weekly Checkpoints (Low-Stakes)
**When:** Mid-week checkpoint
**Format:** Combination of 3-4 questions from current chapter
**Configuration:**
- Allow 2-3 attempts
- Show correct answer but not explanation until after deadline
- Worth 5-10% of final grade
- Quick turnaround feedback (24 hours)

### Strategy 3: Comprehensive Exam (Summative)
**When:** End of Weeks 1-4
**Format:** All 40 questions (randomized question order)
**Configuration:**
- Single attempt
- No time limit recommended (bootcamp pace is intensive)
- Show explanations after grading
- Worth 20-30% of final grade

**Scoring Options:**
- Simple: 1 point/question = 40 points
- Weighted by Bloom's:
  - Remember (×1): 8 points
  - Understand (×2): 24 points
  - Apply (×3): 39 points
  - Analyze (×4): 28 points
  - Total: 99 points (normalize to 100)

### Strategy 4: Practice Bank
**When:** Between chapters, before major assessments
**Format:** Topic-based (students choose topic)
**Configuration:**
- Unlimited attempts
- Immediate feedback with explanations
- No grade tracking
- Use for last-minute review

---

## Performance Benchmarks

### Expected Score Ranges

**Strong Cohort (>80%):**
- 36-40 correct (90-100%)
- Demonstrates mastery of most concepts
- Ready for advanced topics

**Proficient (70-79%):**
- 28-35 correct (70-87%)
- Solid understanding of core concepts
- May need review of specific areas

**Developing (60-69%):**
- 24-27 correct (60-67%)
- Understands fundamental concepts
- Needs targeted support

**Needs Support (<60%):**
- <24 correct (<60%)
- Indicates gaps in foundational understanding
- Recommend 1:1 tutoring or review sessions

### Question-Specific Performance Targets

**All chapters:** Target <30% incorrect rate
**If >30% incorrect:** Indicates need for reteaching

**Watch questions for misconceptions:**
- Q2.4, Q2.10: Git workflow confusion
- Q3.2, Q3.6: Database design principles
- Q4.4, Q4.8: BigQuery optimization strategy
- Q1.4, Q1.6: Memory management in Python

---

## Identifying Learning Gaps

### By Concept:

**Python/SQL (Ch1):**
- If Q1.1, Q1.3 missed: Generator/window function review needed
- If Q1.4, Q1.6 missed: Memory efficiency principles need reinforcement
- If Q1.10 missed: Query optimization strategies need practice

**Git/Docker (Ch2):**
- If Q2.3, Q2.4 missed: Git workflow retraining needed
- If Q2.6, Q2.9 missed: Docker optimization concepts unclear
- If Q2.8 missed: Git security best practices not internalized

**Database Design (Ch3):**
- If Q3.1, Q3.2 missed: Normalization principles need review
- If Q3.4, Q3.8 missed: Index strategy not well understood
- If Q3.10 missed: Query plan analysis needs practice

**Data Warehousing (Ch4):**
- If Q4.2, Q4.5 missed: Schema types/OLTP vs OLAP confusion
- If Q4.4, Q4.8 missed: BigQuery-specific optimization unclear
- If Q4.9, Q4.10 missed: Data warehouse design patterns weak

### By Bloom's Level:

**If many Remember questions wrong:**
- Students may not have attended lectures
- Assign lecture review or video tutorials

**If many Understand questions wrong:**
- Concept explanations unclear
- Use office hours for 1:1 explanation
- Consider creating supplementary resources

**If many Apply questions wrong:**
- Hands-on practice needed
- Assign additional lab problems
- Review real-world code examples

**If many Analyze questions wrong:**
- Critical thinking skills developing
- Assign case studies
- Encourage peer discussion/debate

---

## Student Communication

### Share with Students:

**Before Quiz:**
```
This quiz assesses your understanding of [Chapter X topics].
- Complete by [deadline]
- You may retake up to [X times]
- Use explanations to guide your studying
- Aim for 80%+ for mastery
```

**After Quiz (Feedback Email):**
```
Congratulations on completing the quiz!

Your Score: [X]%
Proficiency Level: [Developing/Proficient/Advanced]

Strong Areas:
- [Topic from questions you got right]

Areas for Review:
- [Topic from questions you got wrong]

Recommended Actions:
1. Review answer explanations for incorrect questions
2. [Specific assignment/resource]
3. Attend [office hours/study group]

Next Steps:
- [Upcoming deadline/assessment]
```

---

## Troubleshooting

### Issue: Students Struggling with Distractors

**Solution:**
- Remind students that all options are plausible
- Teach test-taking strategy: eliminate obviously wrong (none here), then reason through remaining
- After quiz, review why each distractor is tempting

### Issue: High Variance in Scores

**Solution:**
- May indicate students have different background knowledge
- Check if different learning styles need support
- Consider peer tutoring/study groups
- Identify if specific topics need different teaching approach

### Issue: Students Performing Better on Later Chapters

**Solution:**
- This is normal! Building knowledge is cumulative
- Students learning "how to learn" this material
- Celebrate progress
- Don't lower expectations for earlier chapters

### Issue: Students Perform Better on Remember vs Apply

**Solution:**
- This indicates need for more hands-on practice
- Assign lab projects aligned with Apply/Analyze questions
- Use case studies in lectures
- Encourage code review discussions

---

## Customization Options

### Adjusting Difficulty:
- **Remove:** Analyze level questions (Ch1 Q10, Ch2 Q8/Q10, etc.)
- **Add:** More Apply level questions from question bank
- **Simplify:** Use only Remember + Understand for first-time learners

### Creating Targeted Quizzes:
- **By Topic:** Select questions by concept tags
- **By Level:** Select by Bloom's level (e.g., Apply-only practice)
- **By Week:** 5 questions/week for consistent assessment

### Scoring Adjustments:
- **Reduce:** Remove highest difficulty questions
- **Increase:** Weight Analyze questions more heavily
- **Curve:** Add points if whole cohort struggles

---

## Assessment Timeline (Suggested)

### Week 1-2 Schedule:
- **Sunday:** Formative quiz (Ch1 Q1-5)
- **Wednesday:** Checkpoint (Ch1 Q6-10)
- **Friday:** Formative quiz (Ch2 Q1-5)

### Week 3-4 Schedule:
- **Sunday:** Formative quiz (Ch3 Q1-5)
- **Wednesday:** Checkpoint (Ch3 Q6-10)
- **Friday:** Formative quiz (Ch4 Q1-5)
- **Monday (Week 5):** Comprehensive exam (all 40 questions)

---

## Resources for Students

### Study Materials to Provide:
- INDEX.md (quiz overview and navigation)
- QUESTION_ANALYSIS.md (detailed concept breakdown)
- Link to course lecture recordings
- Links to relevant documentation:
  - Python: generator syntax, context managers
  - SQL: window function docs, CTE examples
  - Git: merge vs rebase tutorials
  - Docker: official documentation
  - BigQuery: pricing calculator, query examples

### Recommended Study Approach:
1. Complete chapter quiz once for diagnosis
2. Review answer explanations for incorrect answers
3. Review corresponding lecture/textbook section
4. Review correct answer explanations again
5. Retake quiz to verify understanding

---

## Continuous Improvement

### Feedback Loop:
1. **Analyze Results:** Which questions have >30% wrong?
2. **Identify Cause:** Student knowledge gap or unclear question?
3. **Update:** Revise lecture/quiz explanation or teaching approach
4. **Re-assess:** Test improvement with next cohort
5. **Document:** Track changes for curriculum review

### Questions Needing Review (if performance is poor):
- Q2.4 (rebase safety - commonly confused)
- Q3.2 (2NF violations - abstract concept)
- Q4.4 (partition vs cluster - similar concepts)

### Adding New Questions:
When new concepts emerge:
1. Follow the same format (see examples in each quiz.md)
2. Ensure Bloom's distribution remains balanced
3. Create plausible distractors
4. Include detailed explanations
5. Tag with concept name
6. Update QUIZ_SUMMARY.md with new totals

---

## Contact & Support

For questions about:
- **Quiz content:** Refer to QUESTION_ANALYSIS.md
- **Pedagogy:** Refer to QUIZ_SUMMARY.md
- **Canvas setup:** Contact IT/LMS support
- **Student feedback:** Track in gradebook, use for curriculum planning

---

**Last Updated:** 2026-01-28
**Version:** 1.0
**Status:** Ready for Use
