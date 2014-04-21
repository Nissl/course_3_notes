Notes on my code:

DONE
Don't validate emails - don't limit by length or regex matching.
Do this via 2-step - email them

DONE
Password only triggered on create

DONE
return review.rating if review
don't need nil

DONE
don't preface with "new" in controller
review_params.merge!(user_id: )

don't need to check both not_valid and not_save in revies

Fabricator(:bad_review, from: :review) do
	review_text ""
end

whitespace in review

move message set to 

alice.follow!(bob)
charlie.follow!(alice)

still cumbersome, missing method for easy follow
write the code you wish to have, not what you have today

*could* be using let - not Kevin's preferred style - best just to do all the setup for each test. Looks a little ugly/repetitious, but the goal here is to be able to get in and see what's happening fast if something breaks down the line.