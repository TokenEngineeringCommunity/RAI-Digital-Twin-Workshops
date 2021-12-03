### Questions
1. Can you tweak the error being used by the RAI controller? What happens if the integral error is simply the immediate error? Or if other integration rules are used? like Simpson's rule?
2. Right now, the RAI is being modelled as a PI controller. What is required in order to turn it to a PID one? Are you able to implement a demonstration?
3. How would you introduce events into the past and future? How should someone proceed to include shocks into the extrapolation phase?
4. What behavioural model would you introduce on the model? How it would modify the TokenState object? Are you able to implement it?
    - One simple example would be to simulate the effect of an arbitrary liquidity incentive mechanism. This should modify the TokenState on the `rai_reserve` and `eth_reserve` simultaneously.

### Answers
1. Yes you can tweak the error being used from the controller by modifying s_pid_error. If the integral error is the immediate error then you'd imagine it might not be able to maintain the steady state of the RAI target price once hitting it. 

   The controller would  only looking at the minimization of error between the RAI and target price, but not what it takes to maintain that price. 

   If you use other integration rules you'll get different speeds at which you approach / minimize error of the controller output. For example the Simpson rule will probably converge to error state faster than the Trapezoid rule approximation. Which I think will result in quick price convergence.

2. Well we'd need to add a derivative term and maybe a derivative gain term to the params kd. Additionally change controller_params.csv to have kd term and the initial governance backtesting for the model as well. Additional state variable is needed too. [results](derivative_controller_output_with4_-e08_gain.html) for kd 4-E08 constant. 

3.  You could probably modify the exogenous data to introduce a shock by saying at some time stamp the price will just fall or rebound. Or things that change the outside signal.  The other thing you might do is have a governance event to change the param settings in the past. You could also use the partial state update block to introduce state changes as well or new policies that change params.



4. I would try and see what does it look like when the population is irrational , i'd test the behaviors when the population makes the opposite moves of the convergence, in a way stress testing it. So to do this I believe it's a matter of simply reversing the convergence swap intensity signals, which I think making the opposite moves would be indicative of divergent intensity signals.  My implementation  [results](behavioral_shift_with_csi_divergence.html) [github_repo](https://github.com/zcstarr/reflexer-digital-twin/commit/41146514c1056042107ec1c7b67069fabda7fc33)

4. The alternative one, i'm not sure how to implement. I might introduce a minority/majority type of signal to the behavior which might manifest as an aggregate action that happens when a population is bullish on the sell signal and another set are counter inuitive and run from that.  I read a paper not too long go that used these sets of behaviors to model the NYSE I wonder if these behaviors could be used to model the population of RAI holders. 
	
	Minority + 1 lag, Minority + 2 lag, Majority +1  lag , Majority +2 lag. Where the signal was extracted from some number of recent previous states as people tend to be recent history sensitive. The token states generated would then be a function of these
	population distributions, and the previous token states. 

    Unclear how exactly I might implement this, I think I'd need a deeper understanding of RAI token states and how they manifest.


	
