# **Finding Lane Lines on the Road**

[challenge_problem]: ./challenge_problem.jpg "Problematic output in Challenge video"
[challenge_ok]: ./challenge_problem.jpg "Regular output in Challenge video"

## How the pipeline works

I started with a pipeline very similar to the one presented in the course.
I first convert the input image to greyscale and apply a gaussian blur of size 5.
Then I apply canny edge detection with a `low_threshold` of 50 and
`high_threshold` of 150.
Now crop the output to only include the region of interest.
This is helpful later for drawing the solid lines.
I then apply the hough transform with `min_line_length` of 20
and `max_line_gap` of 40. I found that this helps finding
longer line segments.

To draw the 2 lines for left and right, I first split the lines found in 2 batches.
To decide which side a line segment belongs to, I look at the midpoint.
This is where it comes in handy that I applied the region of interest filter early,
as I can take most of then as they are.

I then convert all line segments into their parameter form, so that I can take a weighted average.
I decided to thow all the one out, where $abs(sin(angle))<0.5$.
This corresponds to a far shallower angle than the lane markings, so this helps clean up some of the falso positives.
It also mitigates a potential division by zero error, but this could also be dealt with differently if necessary.

To average the found parameters, I use the square of the length of the corresponding segments as weight.

## Evaluation

To start off here is an output of the pipeline for the challenge video:
![challenge_ok]

Looking at just this you might assume that the pipeline is fairly robust.
The pipeline throws out horizontal line segments, which helps with the hood of the car.
It also tries to minimize noise by looking at longer line segments with more weight.
This, however is no guarantee that it won't get distracted, as can be seen in the following output:
![challenge_problem]

In this instance, the shadows of the tree create strong lines.
They also make the edge of the lane markings less clear.

The chosen design of just looking at the length does help in reducing noise,
but it is not very good at eliminating outliers like shadows or different tarmak.

## Possible improvements

At the moment I only look at the line length and don't make any further assumptions.
I do know however, that lane markings roughly go towards the horizon and the car is rougly traveling in that direction.
With this knowledge, I could rank the found line segments by their angle as well as their length.
The further away the line segment is from this assimed trajectory, the less likely the match.
