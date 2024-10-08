---
title: MontyHall(UNSW)
categories:
  - Algorithm
date: 2024-10-04 07:24:00
tags:
---

- [introduction](#introduction)
- [problem](#problem)
- [analysis](#analysis)
- [simulation](#simulation)


# introduction

I first meet Monty Hall problem in a korean drama that is called D.P.: A soldier escaped from the military, and applied the essence of Monty Hall problem to make life better, although I think it was not rigorous.

Monty Hall problem is not complicated, it is an only high school level’s statistic problem. But it is a little counter-intuitive.

[Monty Hall on wikipedia](https://en.wikipedia.org/wiki/Monty_Hall_problem)

The description of Monty Hall problem is cited from UNSW COMP9021, there is a nuance between the wikipedia’s description and the context below about **2 goats** OR **1 goat and one empty room**. I would prefer to use `2 goats` later, and please regard the two conditions as an equivalence. Just regard choosing the car as a success.

# problem

A contestant and a host are at a place that gives access to three rooms, whose doors are all closed before the game starts. A car is in one room, a goat in another, while the third room is empty. The host knows what is in each room, while the contestant doesn’t. The contestant is asked to chose one of the doors. Following that choice, the host opens another door, which is that of a room that does not contain the car. The contestant is then asked to either stick to his choice or “switch”, that is, choose the third door (e.g., the door which is neither the one he chose in the first place nor the one that the host opened). The host opens the door as requested by the contestant, who wins what is in the room, if anything. The question is: what is the contestant’s best strategy – to switch or not to switch?

# analysis

The contestant wishes to win the car, not the goat. He has one chance out of three to, in the first place, choose the door of the room that contains the car.

- If he does not switch, he then has one chance out of three to win.
- If he switches then he loses in case the room he chose in the first place contains the car. But he wins otherwise. Indeed, out of both doors that are still closed, one contains the car, so the host has to open the other one (that is, if the room whose door was chosen in the first place contains nothing, then the host opens the door of the room that contains the goat, and if the room whose door was chosen in the first place contains the goat, then the host opens the door of the room that contains nothing). So in both cases, the second door that the host opens following the decision of the contestant to switch is that of the room that contains the car.

Hence by switching, the contestant has two chances out of three to win. Hence it is best to switch.

The car is put average Randomly behind the rooms, so we can use chart to list all the situations.

| Behind door 1 | Behind door 2 | Behind door 3 | Result if staying at door | Result if switching to the door offered |
| ----- | ----- | ----- | ----- | ----- |
| Goat | Goat | **Car** | Wins goat | **Wins car** |
| Goat | **Car** | Goat | Wins goat | **Wins car** |
| **Car** | Goat | Goat | **Wins car** | Wins goat |

For the contestant, at the beginning before the host opened a room without car, he has 1/3 chance to win the car; but later the host intendly chooses the room without car, notebly it is not random. And contestant now has 1/2 chance to win the car of these two room left.

But it is an entire game, we need to calculate the opportunity from the very first beginning. Our intuition of 1/3 and 1/2 is not wrong, but it is counter-intuitive to compute the cumulative chance. The contestant doesnot do temporary decision, which is only 1/3 and 1/2 at those stages. He wants to do the best strategy of the whole game. The interference factor is the host’s choosing intendly.

I’d better to recommand the probability calculation below or simulation to figure it out.

# simulation

We aim to simulate the playing of this game, with either strategy, switching or not, to check that when it is played with the chosen strategy a large enough number of times, then the experimental results support the conclusions of the previous reasoning. More precisely, the user should be asked and express whether he wants to switch and how many times the game should be played. By default, if the user requests to play at most 6 games, then the details of each game (which door the contestant choses in the first place, which door the host opens first, and which door he opens next that determines the outcome of the game), will be output; if the user requests more games to be played, then the details of the first 6 games only will be output. It should be possible to set the default of 6 to another value. In any case, the percentage of times the contestant won the game should be eventually displayed.

**Notebly: Don’t use seed** in a totally random scene. Random function only is enough to simulate a totally random number, and seed is only to avoid the predicted of the random function, because it depends on system’s date or something. But the dependency is a really random number.

```python
# Written by Eric Martin for COMP9021


'''
Simulates the Monty Hall problem.
- A car is hidden behind 3 doors.
- The contestant randomly choses a door.
- The game host opens a door behind which there is no car.
- The contestant then has the option to change her mind and open another
  door.

Prompts the user for the number of times the game is played, and whether
the contestant opts for switching door or not.

Displays the details of the game for the few first games, set to a
maximum of 6 by default, and prints out the proportion of games being
won.
'''


from random import choice, randrange


def set_simulation():
    while True:
        try:
            nb_of_games = int(input('How many games should I simulate? '))
            if nb_of_games <= 0:
                raise ValueError
            break
        except ValueError:
            print('Your input is incorrect, try again.')
    while True:
        contestant_switches = input('Should the contestant switch? ')
        if contestant_switches.istitle():
            contestant_switches = contestant_switches.lower()
        if contestant_switches in {'yes', 'y'}:
            contestant_switches = True
            print('I keep in mind you want to switch.')
            break
        if contestant_switches in {'no', 'n'}:
            contestant_switches = False
            print("I keep in mind you don't want to switch.")
            break
        print('Your input is incorrect, try again.')
    return nb_of_games, contestant_switches


def simulate(nb_of_games_to_display=6):
    nb_of_games, contestant_switches = set_simulation()
    print('Starting the simulation with the contestant', end=' ')
    if not contestant_switches:
        print('not ', end='')
    print('switching doors.\n')
    nb_of_wins = 0
    for i in range(nb_of_games):
        doors = ['A', 'B', 'C']
        winning_door = choice(doors)
        if i < nb_of_games_to_display:
            print('\tContestant does not know it, but car '
                  f'happens to be behind door {winning_door}.'
                 )
        first_chosen_door = doors.pop(randrange(3))
        if i < nb_of_games_to_display:
            print(f'\tContestant chooses door {first_chosen_door}.')
        second_chosen_door = first_chosen_door
        if first_chosen_door == winning_door:
            opened_door = doors.pop(randrange(2))
            if contestant_switches:
                second_chosen_door = doors[0]
            else:
                nb_of_wins += 1
        else:
            doors.remove(winning_door)
            opened_door = doors[0]
            if contestant_switches:
                second_chosen_door = winning_door
                nb_of_wins += 1
        if i < nb_of_games_to_display:
            print(f'\tGame host opens door {opened_door}.')
            print(f'\tContestant chooses door {second_chosen_door}', end=' ')
            print('and wins.\n') if second_chosen_door == winning_door\
                                 else print('and looses.\n')
        elif i == nb_of_games_to_display:
            print('...\n')
    print(f'Contestant won {nb_of_wins / nb_of_games * 100:.2f}% of games.')


if __name__ == '__main__':
    simulate()
```