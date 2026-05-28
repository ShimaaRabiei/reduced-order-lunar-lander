# Reduced-Order LunarLander

This example defines a reduced-order version of the Gymnasium Box2D LunarLander task. The model keeps the translational motion, terrain, leg contacts, main-engine impulse, and official LunarLander terminal logic, but abstracts away the rotational dynamics.

The policy outputs a main-engine command and a commanded attitude reference. The commanded attitude is imposed directly in the reduced model, so the agent does not learn the rotational dynamics during training.

## Reduced model

The reduced action is

```text
a\_R = \[main\_cmd, theta\_star\_norm]
```

where `main\_cmd` follows the continuous LunarLander main-engine convention and `theta\_star\_norm` is mapped to

```text
theta\_star = theta\_limit\_rad \* theta\_star\_norm
```

The baseline setup used here sets `theta\_limit\_deg = 20`.

The reduced observation is

```text
\[x, y, vx, vy, theta\_star\_previous, left\_contact, right\_contact]
```

The previous attitude reference, theta\_star\_previous, is included because the training objective can include a reference-variation cost.

The reduced model imposes

```text
theta = theta\_star
angular\_velocity = 0
```

at each step. The side thruster is not part of the reduced action and is not fired in the reduced model. Translation is affected by the commanded attitude through the direction of the main-thrust impulse.

## Reward and training objective

The task reward follows the official LunarLander shaping terms for distance to the pad, velocity, attitude, leg contacts, main-engine fuel cost, and terminal success/crash reward. Since the side thruster is not part of the reduced action, no side-thruster fuel cost is used in the reduced model.

The training reward is

```text
train\_reward = task\_reward - variation\_lambda \* |theta\_star\_t - theta\_star\_{t-1}| - step\_penalty
```

The step penalty is used to discourage indefinite hovering. It does not replace the task reward and it does not change the official success condition.

## Training and evaluation

Training uses the official LunarLander reset mechanism. Evaluation uses fixed reset seeds so different training runs can be compared on the same initial conditions.



The `variation\_lambda = 0` baseline was trained in two stages: a cold PPO run followed by a warm fine-tuning run with a smaller learning rate and smaller exploration noise.

## Lambda-zero baseline result

```text
variation\_lambda: 0.0
theta\_limit\_deg: 20
step\_penalty: 0.02
evaluation episodes: 333

success\_rate: 0.5225
crash\_rate: 0.1021
out\_of\_bounds\_rate: 0.0060
mean\_task\_return: 139.6360
mean\_train\_return: 125.5874
mean\_variation: 2.9319
mean\_length: 702.4294
mean\_final\_speed: 0.0131
mean\_final\_both\_legs\_contact: 0.7447
mean\_final\_lander\_awake: 0.4775
landing\_candidate\_rate: 0.4474
```

## Repository files

```text
train\_reduced\_order\_lunar\_lander.py
models/lam0\_theta20\_step002\_warm/final\_model.pt
models/lam0\_theta20\_step002\_warm/config.json
models/lam0\_theta20\_step002\_warm/training\_history.csv
results/lam0\_theta20\_step002\_warm/eval\_333\_summary.json
commands/lam0\_training\_commands.md
```

