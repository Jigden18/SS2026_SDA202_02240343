# Reflection on Class and Object Diagrams for the Automated Grading System

## Overview

The Unified Modeling Language helps us understand software systems. It does this in two ways: class diagrams and object diagrams. Class diagrams are like blueprints. They show what attributes and behaviors a group of objects share. Object diagrams show what is actually happening in the system at a point in time. They give a snapshot of the instances in the system and the relationships between them.

## The Relationship Between Class and Object Diagrams

I looked at my Automated Grading System. The class diagram gives the structure. For example, the Professor class has attributes like professorId, name, and email. It also has methods like createAssignment() and overrideGrade(). The object diagram shows instances of these classes with values. So while the class diagram says a Professor has a name attribute, the object diagram shows a professor named 'Dr. Smith'. This distinction matters. It helps move from the general to the specific. I can see what is possible. Then I can see what actually happens when the system runs.

## Key Notations

### Objects and Instance Specifications

Each instance appears as a rectangle. It shows the object name and its attribute values. For instance, the Professor instance shows professorId = 'P100', name = 'Dr. Smith', and email = 'smith@uni.edu'. The plus symbol means the values are public.

### Links versus Associations

In class diagrams, associations become links in object diagrams. In my class diagram, I defined an association between Professor and Assignment. It says one professor can create many assignments. In the object diagram, this becomes a link between instances. The Professor object for Dr. Smith connects to the Assignment object A201.

### Dependency Relationships

A dependency exists when a change in one element may cause changes to another. In both diagrams, the AutoGradingEngine depends on Submission and GradingCriteria. Dashed arrows show this.

### Composition and Aggregation

Composition uses a filled diamond. Aggregation uses a hollow diamond. In my class diagram, I used composition with AuditReport and Submission. It shows that an AuditReport archives Submission objects. The report cannot exist without the submissions it contains.

## Benefits

### Detailed Insight into Relationships

The object diagram shows how specific objects interact. Student John Doe makes Submission Sub99 for Assignment A201. The AutoGradingEngine processes it and produces Grade G88 and PlagiarismReport PR55.

### Implementation Guidance

Concrete attribute values give developers examples. They can see how to instantiate objects and what states those objects can hold.

### Test Case Design

Testers can use the scenario to design integration tests that verify interactions. The concrete values provide expected inputs and outputs for testing.

### Validation of Code Implementation

Developers can check whether their code matches the intended design. If the code creates a Grade object but fails to set the publishedAt attribute, the diagram shows this mistake.

## Practical Application: The Automated Grading System Scenario

The object diagram captures a realistic scenario.

First, Professor Dr. Smith creates Assignment A201 for the OOP Project. The deadline is May 20, 2024. It allows up to 3 attempts.

Second, Student John Doe submits his work as Submission Sub99 on May 19, 2024. The status is GRADED.

Third, the AutoGradingEngine processes the submission. It uses TestBasedCriteria with a threshold of 0.6.

Fourth, this processing produces a Grade of 0.85 (85%). It also produces a PlagiarismReport showing 12% similarity from Turnitin.

Fifth, the grade triggers the NotificationService, which would alert the student.

Sixth, external systems handle publishing assignments, receiving grades, and checking plagiarism.

Seventh, a RegulatoryBody could request an AuditReport. It archives the submission for compliance purposes.

This snapshot tells a complete story of one grading transaction.

## Multiple Object Diagrams

Multiple diagrams can show different states over time. Before submission, the Submission object has status DRAFT. During grading, the Grade object may exist with score = null. After a grade override, the same Grade shows isOverridden = true.

## Limitations

Object diagrams have some limitations. They show only one moment, not the whole process. You need multiple diagrams to show state changes. They show structure and relationships but not the sequence of operations or the algorithms used.

## Conclusion

Using both class and object diagrams gives a complete view of the Automated Grading System. The class diagram answers what structures and relationships are possible. The object diagram answers what is actually happening right now. Object diagrams help with system implementation, team communication, test case design, debugging, and documentation. Creating these diagrams pushed me to think about specific values and interactions. This revealed assumptions and edge cases that the class diagram hid on its own.