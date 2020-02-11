# Exploring React Additional Hooks - Talk

#### Talk goals: 

- Attendants should be able to :
    - reason why those hooks are introduced and when to use them. 
    - what pitfall are when using them improperly. 

#### Talk outline : 

- Talk and speaker introduction. 
- Why did I plan to give this talk. 
- What are react hooks, briefly. 
    - explain what happens to component rendering when calling useEffect to set state.
- mental concepts to change :
    - it has rare cases.
    - useEffect and hooks. 
    - performances.
- Common case of hooks usage. 
- The missing gap that require additional hooks. 
- ... what are those hooks. 
    - useCallback 
        - some reason why not : https://kentcdodds.com/blog/usememo-and-usecallback
        - showcase child re-render for functions. 
    - useMemo: 
        - show an example of high performance calculation. 
        - 
    - useReducer
        - How to use it to slowly re-write classes. 
        - show the car selector using : https://css-tricks.com/getting-to-know-the-usereducer-react-hook/ and reducer animation https://css-tricks.com/understanding-the-almighty-reducer/
        - show case the usage of accessing the previous state, with example from here: https://adamrackis.dev/state-and-use-reducer/
        - show example of useReducer with calculate the tax income form ?? 
        - 
    - useContext
        - Refactor diff from provider to useContext, side to side comparison. 
            - for more examples : https://frontarm.com/james-k-nelson/usecontext-react-hook/
        - reciepe for useContext with useReducer. 
    - useRef 
        - show case a useRenderCounter hook. 
- Hooks definitions and usage. 
- game demo : https://egene.github.io/planet-escape/index.html
- comparison to what it's been used in Class component. 
- Benefits in performances.  
    - quote "If you aren’t measuring, you can’t even know if your optimizations are better, and you certainly won’t know if they make things worse!" 
- Demo of updating a class component to a hooks.
- maybe to multiple hooks.
    - can be processing large data in table. 
    - 
- comparison in code after refactoring.
- recap and additional usage for those hooks. 
- Questions. 
