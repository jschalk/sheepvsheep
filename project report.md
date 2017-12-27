# sheepvsheep
Netlogo Agent Based Model for SYSC 525 (taught by Dr. Wayne Wakeland) by Joachim Schalk

Below is an edited version (pictures omitted) of the report on the model. It was written for the intended audience of Wayne Wakeland and Dale Frakes and reflects my understanding of their preferences. 

Project was completed in Netlogo 6.0. Primary references for troubleshooing was netlogocommons.com, stackoverflow


Problem statement

Can simple rules produce complex political behavior? And does allowing sheep to eat some of their neighbors produce complex behavior such as kingdoms with territories, armies, and cyclical history? 


Summary

The Sheep-Wolf depredation model is a classic intuitive Netlogo model. But what if instead of wolves eating sheep like in biology the sheep eat other sheep like in politics? I followed this reasoning as far as I could and maneuvered around coding limitations (mostly my own but also some of Netlogo’s) to try to get to my goal of territorial kingdoms and perhaps even armies. Unfortunately those did not emerge. But there were a number of interesting features in the final model. The most obvious being the unplanned emergence of complete population stability during growth and at max population. 
Building the model followed three major themes
1. Changing some of the basic rules of the model per intuition. This included things such as changing grass-sheep connection so that multiple sheep could eat from one field and reducing roaming to those that were hungry. 
2. Getting the model to work. There were nine major saved models that explored how to get a stable model. The most difficult part was attempting to get sheep to prey on other sheep. 
3. Incorporating Moore neighborhoods. The prey mechanic was abandoned and the concept of subjugation by energy transfer was established. Then interesting dynamics and emergent properties appeared.


Background

The Wolf Sheep Predation model is a simple and fun model found in the Netlogo library that is automatically downloaded with the application. The model is so well regarded that even in that limited library that there are multiple variations that explore different ideas. The dynamics of the model are pretty simple, the wolves and sheep are two breeds of turtles, they roam the world. If the sheep find grass they eat it. If the wolves find sheep they eat them. The interesting question is how populations of wolves and sheep react to each other and how can they stabilize. There are multiple parameters that can be changed and some have a large effect and others a small one. The model often results in extinction. 
What I found interesting (that no one else seemed to find interesting) was that if the wolves went extinct first the sheep would quickly come into balance with the grass and a stable population cycle would emerge. The sheep would eat the grass, the grass would all die, the sheep would start to die, the grass would grow back and the sheep would recover from a population crash. There was something there. It just needed to be explored. 
Screenshots of stable sheep/grass populations from the Netlogo default library.


Process of Building the Model

As much as possible the ODD protocol from Railsback and Grimm was followed. The most important thing they mention over and over again is to know your purpose and stick to it. Even though I read that many times and believed it many times, many times I found myself following programming tangents that were not serving that purpose. To get on track I had to simplify my operating procedure to the following:

Try get an interesting model. Solve little problems and then stop and see if you’re getting there.

I started by taking the model as given by Netlogo and tweaking the parameters to identify what did what to the populations. Increasing the initial size of wolves didn’t seem to do much. Often the results were counter intuitive such as increasing the amount of food wolves receive from successfully killing prey, deceasing total wolf population and increasing sheep population. 
Next I started turning the model towards the goal of sheep kingdoms. There were many changes to the model and it is difficult to identify all of the them even with the notes (though the notes could have been much better) so I will mention a couple of the processes that I remember the best. 

Getting large populations of sheep to survive on shared patches of grass. 
Originally grass is a binary status of green or brown. Instead I gave each patch a g-energy property (grass energy) and gave it a max of 100. Whenever g-energy was at or less then 20 the patch displayed as brown. The g-energy continually increased. Naturally this meant that sheep needed to consume not the binary green/brown properties but a portion of the g-energy. For the conversion from grass energy to sheep energy the general proportion of 3-1 was applied. If the sheep harvested 3 g-energy it received 1 sheep energy.  Below is some of the simple grass logic. Eventually I incorporated the idea that if the neighbor patches had a lot of grass the grass should grow faster.

	to grow-grass
	  if g-energy < 100 [
	    ifelse sum [ g-energy ] of neighbors > 100
	      [ set g-energy g-energy + 5 ]
	      [ set g-energy g-energy + 1 ]
	  ]
	  ifelse g-energy <= 20 [ set pcolor brown ] [ set pcolor green ]
	end

All this programming required learning new functions and methods. It also required debugging and testing. The initial runs were wildly chaotic with giant spikes in sheep population resulting in massive wolf populations covering the entire map and then all the wolves killing off all the sheep. 

To get kingdoms sheep got to stop roaming randomly. 
The roaming dynamic were changed so that instead of sheep randomly moving across the map the moved when they were hungry. This was defined by low energy. The idea behind this is that in civilization people only move when they have to because moving requires huge resources and is risky. This resulted in the emergent phenomenon of large patches of grass being untouched as sheep stayed where they were and killed off all the grass, then they would move across the map in a migration eating untouched grass until a swarm of wolves would come and eat most of them. The grass would regrow untouched until the next migration came over. Cool stuff. 
To continue this process the same logic was applied to wolves. This resulted in large groups of wolves sitting and waiting for sheep to come to them. Walls of predators waiting for migrating sheep. When they get hungry they move and die off most of the time. This seemed to help stabilize wolf populations. 

To get sheep to eat each other how about getting wolves to eat each other?
For Models 3 – 7 wolves were set to hunt other wolves. They ended up not eating each other because then they would eat themselves immediately. I set sheep maximum energy to 1,000. This resulted in widely chaotic behavior that was stable. The wolf populations where larger than the sheep populations.
	
Set up a dynamic where 2 breeds of (blue and black) wolves and 2 breeds of sheep (white and yellow) exist. The wolves only move when they are hungry. This results in interesting emergent behavior of packs of wolves moving after and then killing packs of sheep. (It was also learned that the internet has an endless supply of blue wolf images. Or basically any color wolf. )

For Model 8  I made it so that wolves lose a lot of energy by roaming. This resulted in short-term stable and entertaining wolf and sheep population movements with tight cycles of sheep and wolves population sizes. But the population dynamics normally resulted in population extinction within a couple cycles. Eventually I had to resort to automatically creating a new wolf of each breed once during each tick. This was a programming tangent that should not have been followed. 

Model 9.2: The blue wolves and black wolves can eat each other now. The blue wolves eat yellow sheep and the black wolves eat the white sheep only.

Model 9.3: The blue wolves and black wolves engage in war due to an error in reproduction. They reproduce three times their energy. There are 130,000 wolves after 100 ticks.
It was at this point I recognized that I needed to start again from the beginning because the model was going on the deep end.

Getting back on track.
Kept the grass dynamics and the roaming logic. Figured out how to eliminate suicide basically by eliminating the prey method. Wolves were eliminated and no longer exist from the beginning. The goal was eventually to allow anyone to be a wolf. Kept all grass logic, roaming logic. 

The way to “prey” on other sheep (that was eventually settled on instead of other potentially good options) was the Netlogo neighborhood methods. For each sheep all it’s Moore neighbors who are weaker are put on a list. Everyone on that list loses energy. So every weak sheep is exploited by every stronger sheep. Here’s the code. The ‘winner-takes’ variable is a parameter that can be changed by slider. 

	to eat  ; wolf procedure
	  set my-energy energy ; set global variable to current agent energy
	  ;attempt to prey on weaker neighbors
	  ; define prey as all agents in the not von-nuemann neighborhood
	  let prey sheep1-on neighbors  
	  set prey prey with [ energy < my-energy ] ; set the prey list to only weaker ones
	  ; all the weaker ones if you energy
	  set energy energy + (winner-takes * (count prey) / 2) 
	  ; if the sheep is king of the neighbors then set as black. Else set as white
	  ask prey [ set energy energy - winner-takes ] ; all the weaker ones are made weaker
	  ifelse count prey > 1 AND count prey = sum [count sheep1-here] of neighbors 
	    [ set color black ]  [ set color white ]
	  ifelse energy > 100 [set energy 100] [ eat-grass ] ; if sheep still hungry eat grass.
	end

Building the model was a chaotic story. If I did it again I would have taken my time and refused to follow rabbit holes that wasted time. Eventually I became trained to do that. While writing this report there have been multiple changes I’ve wanted to make and test out. But I’ve only gone away to try them out twice! That’s pretty good compared to how often I wanted to.


W1 Model Summary: No oppression yet.

This model has no wolves. The sheep roam only when they are hungry. The grass is overeaten and then migrations happen, unless there is no grass in which case there is a large die-off. The sheep that survive come and eat all the new grass.  The population yo-yos up and down but is stable. Extinction does not seem possible in the long term. 

Emergent behavior includes group migration, group die off, wanders sometimes succeeding but mostly dying. Learned to overcome coding challenges. 


W2 Model Summary: Oppression!

The oppression dynamic is introduced. (It’s explained above with it’s code at the end of the ‘Building the Model’ section) The logic of the oppression is based on the local Moore neighborhood of the sheep and a strong-takes-the-weak principle. The results were pretty striking. 

Migration completely stopped. This makes sense because those who wander around are only wandering because they are weak and the weak get killed off by everyone stronger then them.

There was often a single ‘king’ in each neighborhood and they seemed to rarely move. Kings who wandered because of dead grass became subjects to others. Mostly though they had so much energy they would sit and wait for the grass to grow and then hatch underlings. If ‘war’ mode is turned off the king sheep have so much energy they wander without problems for a long time.

There was a complete stability of population numbers. In the scenario above the ‘war’ option was engaged from the beginning and then turned off. The population grew slowly and leveled off without any real fluctuations. This was the most surprising emergent property.  


W3: Stability

The goal was to get political territories given only local neighborhood rules. To do this the energy of all sheep was capped to 100. The amount that all weaker sheep in the neighborhood were oppressed was changed from 2 energy to twice a changeable parameter. So the weak still get eaten a lot (got to keep the poor down) but how much can be tested by making the amount of energy subjecting should take changeable by slide.

No large kingdoms. Extremely stable populations (see populations graph with a middle section where war was turned off). No armies.


W3 Model testing. 

BehaviorSpace continually returned errors for the last few days of attempts. Unfortunately I’ve been unable to troubleshoot the problem. The issue I most wanted to test whether increasing amount of energy taken by the stronger sheep had a power law or linear effect on population levels. Here’s what I was able to get from my own manual testing. This is from the W3 model. 

Winner-takes    2     3     4     5
Sheep           1180  1016  928   841
Kings           247   205   161   133

Based on this limited data there is a power law relationship. This makes intuitive sense because kings collect energy from their subjects even if they don’t need the energy. That excessive collection results in greater lost energy from the entire system. But if that lost energy wasn’t lost it would support more sheep (because all stronger sheep oppress weaker sheep). Clearly though it would be nicer to have more data. 


Problems Encountered

The biggest problem encountered was the lack of immediate emergent results wanted. This is probably normal. My solution was to keep trying. And try to be happy with what did work.

The rest of the problems also seemed pretty normal. Code troubleshooting seemed to dominate time spent. It was difficult to find any related literature because all referenced wolf-sheep models follow the biological instead of political train tracks. Searching for relevant models became paralysis because of excessive information. Perhaps if I had found a blog with relevant posts I would have been able to follow the list of models and their references. Unfortunately the Hobbesian concept of the people being the body politic of king does not seem to have translated to much ABM research. My solution was to dedicate more time to testing different avenues of programming ideas. 


Next Steps

Next steps include a Sugerscape like approach of adding the desired emergent quality by finding neighborhood based rules that push that. The most interesting question is how to create larger contiguous kingdoms. One approach could be to expand how far kings can see and thus expand how far they can oppress but that’s not in keeping with the ABM principle of giving each agent a limited scope of information. A better solution would be to create a influence peddling dynamic. Of all the sheep being oppressed some would be oppressed more while a smaller group would be oppressed less. In return those favored would be given power to oppress outside the king’s Moore neighborhood. Which might only be three spaces if the influence peddler is on the side or 6 spaces if they are on a corner. 

There are a range of parameters to study and explore including expanding how much grass energy can be generated per tick and in total per patch. Introducing seasons when nothing grows, thus requiring food storage. Then perhaps food trade. There are endless possiblities. 


Conclusion & France

The goal was to have complex politics emerge from basic oppression. I started with the basic Sheep-Wolves-Predation model from the Netlogo library and after many dead-ends eventually created an interesting model. The process of building involved difficult problems and lots of intuition. The model continually developed and started from an unstable chaotic mess to something that resembles a working system. The biggest step was getting better coding tools for allowing sheep to eat their neighbors. The model successfully existed without wild fluctuations and extinctions and showed emergent properties some of which were not predicted. The coolest and most obvious property is that the population became very stable whenever oppression was engaged and population went back to boom-bust otherwise.

The population stability property has recently made me think of France. The story goes, as told around the table with my French family, that between about 1400 and 1800 the population of France pretty much just leveled off. France was the great power in Europe and as such it should have been rich enough to gain population and thus strength. But there were no population booms. Why? Well the other thing we know about France during this time period is that the nobility loved to party and to pay for the party they horribly oppressed their workers. Perhaps I have found the answer inadvertently!



Thanks for reading. I appreciate the support both of you provided throughout the quarter.
