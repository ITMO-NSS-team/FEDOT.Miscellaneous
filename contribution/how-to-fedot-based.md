## How to create Fedot-based solution

This tutorial describes the main aspects of Fedot-based solutions' development.

The classes of problems that Fedot can solve are listed in the [specification](/FEDOT.Docs/contribution/how-to-embed).

If you want to use Fedot to solve the supported problem and it can be done using existing API and core logic:
1. Analyze the examples from "cases" and "examples" folders
2. Configure models repository if needed
3. Choose the appropriate composer and run the search.
Example: [credit scoring example](https://github.com/nccr-itmo/FEDOT/blob/master/cases/credit_scoring_problem.py).

If you want to use Fedot to solve the supported problem and the modifications of the core logic or API are required:
1. Create a fork of the Fedot repository (or branch inside it), add your modifications, create the pull request.
2. If you have trouble with the desired modifications, create an issue and we will be glad to help. 
Example: any branch in the Fedot repository with prefix "feature-"

If you want to use some components of the Fedot for the non-supported problem:
1. Create a new repository
2. Add the Fedot as a submodule
3. Import its classes and function to implement your functionality.
4. Create an issue to analyze the ways of integration the described functionality into the core.
Example: Neural Architecture Search using Fedot (see nn-structure-optimisation branch or [separate repository](https://github.com/ITMO-NSS-team/nas-fedot))

