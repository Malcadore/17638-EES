# Design Element: State Machines


Relevant Links
- required literature reading

## Types of State Machines
- HSM - Heirarchichal State Machines
- FSM - Finite State Machine

## Background
- What are state machines
- Linking state machines to the behavior of contolled objects
- Linking state machines to events and time
- Processes, Transitions, and Constraints

## Design Guidance
- Changes in enviornment state
- Needed behavioral changes in the application
    - Example: 

## Elements in a State Machine
- Activities
- State Transitions

## Modeling State Machines
- Modeling techniques for State Machines
- State Diagrams, State Charts
- UML
- SysML


## Implementing State Machines

##  Conditional Statements
 
    ```C
    // Define states
    typedef enum {
        STATE_IDLE,
        STATE_RUNNING,
    } State;

    typedef enum {
        EVENT_START,
        EVENT_STOP
    } Event
    
    event = getEvent();

    switch (currentState) {
    case STATE_IDLE:
        if (event == EVENT_START) {
            // Transition to next state
            currentState = STATE_RUNNING;
        }
        break;
    case STATE_RUNNING:
        if (event == EVENT_STOP) {
            // Transition to idle state
            currentState = STATE_IDLE;
        }
        break;
        // Add more states as needed
    }
    ```

    Discussion
    Conditional state machines are the simplest to implement and maintain.  There are no fancy tricks with referncing or structs that need to be coded.  The switch/case block is typically located in a processing loop and executed frequently.  Placing code that queries external conditions can be a matter of taste, being either inside the control loop, the switch case block, or elsewhere, but the point to note is that collecting variables or querying an RT Image is going to be a design concern.  For relatively small state machines, the switch is very maintainable, but when lots of states or transition events have to be designed, then it becomes problematic to update.  This is due to being constrained to one block of code or relocating case code into functions.  Care has to be taken refactoring switch case statements to avoid maintenaince problems.

    * Pros
        - Simple, with no pointers, easy to maintain when small

    * Cons
        - Quickly becomes a maintenaince problem with lots of states 


## Lookup Tables


    ```C
    #include <stdio.h>

    // Define states
    typedef enum {
        STATE_RED,
        STATE_GREEN,
        STATE_YELLOW,
        STATE_MAX
    } State;

    // Define events
    typedef enum {
        EVENT_TIMER_EXPIRED,
        EVENT_MAX
    } Event;

    // Function prototypes for state actions
    void redAction(void);
    void greenAction(void);
    void yellowAction(void);

    // State transition structure
    typedef struct {
        State currentState;
        Event event;
        State nextState;
        void (*action)(void);
    } StateTransition;

    // State transition table
    StateTransition stateTable[] = {
        {STATE_RED, EVENT_TIMER_EXPIRED, STATE_GREEN, greenAction},
        {STATE_GREEN, EVENT_TIMER_EXPIRED, STATE_YELLOW, yellowAction},
        {STATE_YELLOW, EVENT_TIMER_EXPIRED, STATE_RED, redAction}
    };

    // Current state
    State currentState = STATE_RED;

    // State action functions
    void redAction(void) {
        printf("Red Light\n");
    }

    void greenAction(void) {
        printf("Green Light\n");
    }

    void yellowAction(void) {
        printf("Yellow Light\n");
    }

    // Function to handle events
    void handleEvent(Event event) {
        for (int i = 0; i < sizeof(stateTable) / sizeof(StateTransition); i++) {
            if (stateTable[i].currentState == currentState && stateTable[i].event == event) {
                stateTable[i].action();
                currentState = stateTable[i].nextState;
                break;
            }
        }
    }

    int main() {
        // Simulate timer events
        for (int i = 0; i < 6; i++) {
            handleEvent(EVENT_TIMER_EXPIRED);
        }
        return 0;
    }

    ```
    - Explanation
        State and Event Definitions: Enumerations are used to define the states and events.
        State Transition Table: An array of StateTransition structures maps the current state and event to the next state and the action to be performed.
        State Action Functions: Functions that define the actions for each state.
        Event Handling: The handleEvent function iterates through the state transition table to find the matching state and event, performs the action, and updates the current state.
        Main Function: Simulates timer events to demonstrate the state transitions.

    - Pros
        - State transition table is easily defined
        - Functions are mapped to actions that take place on transitions
    
    - Cons
        - Event Handling loop must iterate over the entire state space
        - Handling non-deterministic FSM's in a generic way complicates the example.  
        - Handling more than one event or more than one possible transition makes it also more complicated.


- State Pattern

    ```C
    #include <stdio.h>

    // Forward declarations of state structures
    typedef struct State State;

    // Function pointers for state actions
    typedef void (*StateAction)(void);

    // State structure
    struct State {
        StateAction enter;
        StateAction execute;
        StateAction exit;
    };

    // Function prototypes for state actions
    void idleEnter(void);
    void idleExecute(void);
    void idleExit(void);
    void selectingEnter(void);
    void selectingExecute(void);
    void selectingExit(void);
    void dispensingEnter(void);
    void dispensingExecute(void);
    void dispensingExit(void);

    // State instances
    State idleState = {idleEnter, idleExecute, idleExit};
    State selectingState = {selectingEnter, selectingExecute, selectingExit};
    State dispensingState = {dispensingEnter, dispensingExecute, dispensingExit};

    // Current state pointer
    State* currentState = &idleState;

    // State action functions
    void idleEnter(void) {
        printf("Entering Idle State\n");
    }

    void idleExecute(void) {
        printf("Idle State: Waiting for selection\n");
        // Simulate event to transition to selecting state
        currentState = &selectingState;
        currentState->enter();
    }

    void idleExit(void) {
        printf("Exiting Idle State\n");
    }

    void selectingEnter(void) {
        printf("Entering Selecting State\n");
    }

    void selectingExecute(void) {
        printf("Selecting State: Making a selection\n");
        // Simulate event to transition to dispensing state
        currentState = &dispensingState;
        currentState->enter();
    }

    void selectingExit(void) {
        printf("Exiting Selecting State\n");
    }

    void dispensingEnter(void) {
        printf("Entering Dispensing State\n");
    }

    void dispensingExecute(void) {
        printf("Dispensing State: Dispensing item\n");
        // Simulate event to transition back to idle state
        currentState = &idleState;
        currentState->enter();
    }

    void dispensingExit(void) {
        printf("Exiting Dispensing State\n");
    }

    // Function to update the state machine
    void updateStateMachine(void) {
        currentState->execute();
    }

    int main() {
        // Initial state entry
        currentState->enter();

        // Simulate state machine updates
        for (int i = 0; i < 3; i++) {
            updateStateMachine();
        }

        return 0;
    }
        ```

    - Explanation
        State Structure: The State structure contains function pointers for the enter, execute, and exit actions.
        State Instances: Instances of the State structure are created for each state (Idle, Selecting, Dispensing).
        State Action Functions: Functions that define the actions for each state.
        State Transitions: The execute function of each state simulates events that cause transitions to other states by updating the currentState pointer and calling the enter function of the new state.
        State Machine Update: The updateStateMachine function calls the execute function of the current state.
        Main Function: Initializes the state machine and simulates state machine updates.

    - Pros
        - Easier to manage state actions
        - More modular than conditional state machine implementations
        - Transitions defined inside states gets around lookup table limitations

    - Cons
        - Use of pointers can be prohibited by safety standards

## Discussion of Types


## More on State Machines

https://www.youtube.com/watch?v=NTEHRjiAY2I


