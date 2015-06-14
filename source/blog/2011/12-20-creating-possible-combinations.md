@{
    Layout = "post";
    Title = "Creating possible combinations from a list of lists in F#";
    Date = "2011-12-20T17:41:00";
    Tags = "f#";
    Description = "How to find all possible combinations of multiple lists in F#.";
}

Just a short post after I came across an interesting problem while learning F# and couldn’t find any great resources to solve what seems to be quite a generic problem.

The scenario is:
1. You have a list of lists where each of the inner lists contains 1 or more element.
2. You want to find all possible combinations where you take precisely one element from each inner list.

The example of where I found this problem is calculating the possible value of a hand in blackjack:

- You have a set of cards
- Each card has a value
- Ace has both the value 1 and 11 at the same time
- You might be holding more than 1 ace

Therefore we want to be able to calculate the possible values the card could be determined to have.

We represent the value of each card as a list. The five of hearts is `[5]`, the jack of clubs is `[10]`, and the ace of spades is `[1;11]`. If you hold all three of these cards in your hand you can either make 16 or 26.

We would represent this hand as `[[5];[10];[1;11]]` and would ultimately aim to get our results in the form `[16;26]`. However as an intermediate step we want to transform our hand into a list of possible card values such as `[[5;10;1];[5;10;11]]` which we can then simply sum to get our result.

Here’s the function that recursively expands all combinations, returning a list of each combination of items.

    let rec combinations (l) =
        match l with
        | [] -> []
        | h::[] -> h |> List.map (fun opt -> [opt])
        | h::t ->
            combinations t
            |> List.map (fun tOpts ->
                h |> List.map (fun hOpt -> hOpt ::tOpts))
            |> List.concat

Essentially, a quick line by line run through is:

1. To cover all cases, if this is called with an empty list, just return an empty list. This is not used in the recursion.
2. If we are looking at the last list of options then return each option as a new list containing only itself.
3. If we are in the middle of the list (or equally the start) then get the combinations of the tail option lists giving us a list of combinations.
4. For each combination,:
  1. For each option in the current set of options
    1. Add the option to the beginning of each of the tail combinations
5. Flatten the list of tail combinations of head options to outcome list to a list of outcome lists.
