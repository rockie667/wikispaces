---
title: Monopoly_Release_3_Player_Pays_Rent
---
As a player, I pay rent when I land on a Property that is owned by someone else.

### User Acceptance Tests
* Land on a Property owned by other player, player pays rent to owner. Player's balance decreases by rent amount. Owners balance increases by rent amount.
  * If landing on Railroad, rent is 25, 50, 100, 200 depending on how many are owned by owner (1 - 4).
  * If landing on Utility and only one Utility owned, rent is 4 times current value on Dice.
  * If landing on Utility and both owned (not necessarily by same Player), rent is 10 times current value on Dice.
  * If landing on Real Estate and not all in the same Property Group are owned, rent is stated rent value.
  * If landing on Real Estate and Owner owns all in the same Property Group, rent is 2 times stated rent value.

{% include nav prev="Monopoly_Release_3_User_Stories" %}