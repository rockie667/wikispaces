---
title: sbe.WorkItemsInSchedule
---
## Before
{% highlight smalltalk %}
Feature: Handling Scheduling Conflicts
  As an operator I want to make sure feature conflicts are managed by an appropriate policy.

  Background:
    Given a system with no active work items
    And a work item named I1 scheduled to start at 10:00, last for 15 minutes, and use R1
    And a work item named I2 scheduled to start at 10:10, last for 5 minutes, and use R1
    And a first one wins conflict resolution approach

  Scenario: Nothing going on
    When now is 9:59
    Then there should be no active items

  Scenario: One item active
    When now is 10:01
    Then I1 should be active

  Scenario: Conflict Resolved
    When now is 10:10
    Then I1 should be active
    And I2 should be blocked

  Scenario: Delayed Start
    Given now is 10:00
    When now is 10:16
    Then I1 should be completed
    And I2 should be active

  Scenario: Delayed finished
    Given now is 10:00
    And now is 10:15
    When now is 10:21
    Then I2 should be completed
{% endhighlight %}

## After Removing All Extra Time "Movement" Steps
{% highlight smalltalk %}
Feature: Handling Scheduling Conflicts
  As an operator I want to make sure feature conflicts are managed by an appropriate policy.

  Background:
    Given a system with no active work items
    And a work item named Megatron_Torso scheduled to start at 10:00, last for 15 minutes, and use 3d_printer_1
    And a work item named Megatron_Head scheduled to start at 10:10, last for 5 minutes, and use 3d_printer_1
    And a first one wins conflict resolution approach

  Scenario: Nothing going on
    When now is 9:59
    Then there should be no active items

  Scenario: One item active
    Given now is 9:59
    When the time becomes 10:01
    Then Megatron_Torso should be active

  Scenario: Conflict Resolved
    When now is 10:10
    Then Megatron_Torso should be active
    And Megatron_Head should be blocked

  Scenario: Delayed Start
    Given now is 10:00
    When the time becomes 10:16
    Then Megatron_Torso should be completed
    And Megatron_Head should be active

  Scenario: Delayed work item finishes late
    Given now is 9:59
    When the time becomes 10:21
    Then Megatron_Head should be completed
{% endhighlight %}

## After Gojko's Twitter Comment
It's amazing what 140 characters can accomplish. He did it in fewer than that.

{% highlight smalltalk %}
Feature: Handling Scheduling Conflicts
  As an operator I want to make sure feature conflicts are managed by an appropriate policy.

  Background:
    Given an empty schedule
    And a work item named Megatron_Torso scheduled to start at 10:00, last for 15 minutes, and use 3d_printer_1
    And a work item named Megatron_Head scheduled to start at 10:10, last for 5 minutes, and use 3d_printer_1
    And a first one wins conflict resolution approach
    And the business time is 9:59

  Scenario: Nothing going on
    Then there should be no active items at 9:59

  Scenario: One item active
    Then Megatron_Torso should be active at 10:01

  Scenario: Conflict Resolved
    Then Megatron_Torso should be active at 10:10
    And Megatron_Head should be blocked

  Scenario: Delayed Start
    Then Megatron_Torso should be completed at 10:16
    And Megatron_Head should be active

  Scenario: Delayed work item finishes late
    Then Megatron_Head should be completed at 10:21
{% endhighlight %}
