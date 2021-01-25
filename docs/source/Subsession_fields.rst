Subsession Fields
=================

Here is an overview for all subsession fields.::

    sum_saved_emission = models.FloatField(initial=0)

- Checks if a player is a bot (See Explanation in player Class)
- If player is NOT a bot then the players total saved emission of the last round gets added to the sum_saved_emission