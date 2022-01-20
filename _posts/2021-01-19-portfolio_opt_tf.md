---
layout: default
title:  Portfolio Optimisation with Tensorflow and D3 Dashboard
description: Portfolio Optimisation with Tensorflow and D3 Dashboard
date:   2021-01-19 00:00:00 +0000
permalink: /portfolio_opt_tf_d3/
category: Finance
---
## Experimenting with Portfolio Optimisation with Tensorflow | D3 Dashboard

For the heck of it, I just wanted to try to see if I could build a investment portfolio optimise using tensorflow.js, running right inside the browser. After all, portfolio optimisation relies on linear algebra, which tensorflow is well suited for.
 
For those who are not familiar, portfolio optimisation is a key step in asset allocation decisions. 

If you invest in a fund, or use one of those new fangled robot investment advisor services, there’s a very high chance that your investment portfolio is being built using portfolio optimisation techniques.

There are a wide range of techniques used for portfolio optimisation and some can be fairly complex. However, the general steps involved are as follows - 
1. Select the asset classes (e.g. equities, bonds, gold) that you would like to invest in.
2. Get hold of a time series of the prices of these assets.
3. Compute the means, volatilities and correlations of these assets.
4. Think about whether there is a maximum (floor), or minimum (ceiling) proportion of each of these assets you want in your portfolio.
5. Optimise the portfolio by either minimising the volatility of the portfolio, or maximising the Sharpe Ratio. The Sharpe Ratio is the return per unit of risk. Obviously you want to maximise this - the higher the return per unit of risk, the better a deal you are getting.

Steps 1-3 were covered in some of my earlier posts, such as [this][1] and [this][2].

I wanted to be able to fetch data direct via API from a free source online and compute the means, volatilities and correlations, but now that Yahoo Finance and Google Finance APIs aren’t really working, my options were quite limited.  So I’ll just cover step 4 and 5 in this post.

I’ll show you how to create an app (which runs right in your browser, no need for any server!) which can do the following - 
- **Compute Sharpe** - Helps compute the Sharpe Ratio (return/vol) of the return and vol figures entered for each asset
- **Generate Efficient Frontier** - Generate thousands of possible portfolios randomly, and compute the respective returns and volatilities and plot them out. The nice little curve on the left hand side of the chart is our efficient frontier - the set of optimal portfolios that offers either the maximum possible expected return at each level of risk or the lowest possible risk at each level of expected return. (Note: this takes a while as we are literally testing out all possible combinations of portfolios, one by one.)
- **Optimise Portfolio** - Find the best possible portfolio based on the returns, volatilities and correlations provided with matrix operations and tensorflow.js’ Adam optimiser. (Note: This is faster than generating all possible combinations of portfolios, but still takes some time. The progress bar (the orange line) will show you how the training is progressing.)

**Play around with the app [here][3].**

Right at the top of the app is the form which you can use to enter the means, volatiles and correlations that you computed from your own data.  Letting you enter your own data makes sense, as it’s common to use expected/estimated (forward looking) figures rather than relying only on historical figures.

\<\>

The plots in the dashboard show (from left to right) the efficient frontier, the evolution of the Sharpe Ratio during the optimisation process, and the optimised asset allocation.

\<\>

First, the update function gets us the inputs from the form. The code in the function is quite repetitive so I shall just extract the first few lines.
```
function update(){

    // Fetch data from forms
    eqret = $('#eqret').val();
    bondret = $('#bondret').val();
    goldret = $('#goldret').val();
    cashret = $('#cashret').val();
    
    ...
    
    // Save inputs into arrays
    assetRets = [[eqret],
                [bondret],
                [goldret],
                [cashret]
                ];

    assetVols = [[eqvol],
                [bondvol],
                [goldvol],
                [cashvol]
                ];

    corrs = [corr1,
                    corr2,
                    corr3,
                    corr4];

    numAssets = +assetRets.length;


    // This is the variable that we will be trying to get through the optimisation
    asset_weights = tf.variable(tf.div(tf.ones([4,1]), numAssets));

    lossArray = [];
}
```
Next, we have the predict function, which holds the equations that we use to compute the portfolio return, volatility and the Sharpe Ratio. We negate the Sharpe Ratio as the higher the Sharpe Ratio the better, but our optimiser is searching for the minimum.
```
function predict(){
    return tf.tidy(()=>{
    // Normally, we take weightsT*covar*weights
    // Here, we take weightedvolsT*corr*weightedvols
    const weighted_vols = asset_weights.mul(assetVols);
    const vol = tf.sqrt(weighted_vols.transpose().matMul(corrs).matMul(weighted_vols).squeeze());
    const returns_sum = asset_weights.mul(assetRets).sum();
    const sharpe = tf.neg(tf.div(returns_sum,vol));
    // For those not familiar, the Sharpe ratio shows us how much returns we are getting per unit of risk
    // We negate the number as the higher the better, but our optimise is searching for the minimum
    return sharpe;
    });
};
```

Now, we set up the constraints. This had me scratching my head for a while. The comments within explain how they work.
```
function constraints(){
    // All the constraints that we are applying
    // It looks complicated, but all we are doing is resetting the weights at each cycle
    // Weights that breach the constraints will be set to 0, 1, or the preset floors or ceilings
    const floors = [[flooreq],[floorbond],[floorgold],[floorcash]];

    const mask_wts_less_than_floor = tf.greater( floors, asset_weights )
    const floors_ = asset_weights.assign( tf.where (mask_wts_less_than_floor, tf.tensor(floors), asset_weights) );


    const ceils = [[ceileq],[ceilbond],[ceilgold],[ceilcash]];

    const mask_wts_more_than_ceil = tf.greater( asset_weights, ceils )
    const ceilings_ = asset_weights.assign( tf.where (mask_wts_more_than_ceil, tf.tensor(ceils), asset_weights) );

    const mask_wts_less_than_zero = tf.greater( 0.0, asset_weights )
    const zero_floor = asset_weights.assign( tf.where (mask_wts_less_than_zero, tf.zerosLike(asset_weights), asset_weights) );

    const mask_wts_more_than_one = tf.greater( asset_weights, 1.0 )
    const one_ceil = asset_weights.assign( tf.where (mask_wts_more_than_one, tf.onesLike(asset_weights), asset_weights) )

    // This is a little different
    // Here we make sure the sum of weights = 1 by scaling them down
    const result_sum = tf.sum(asset_weights)
    const reset_equals_one = asset_weights.assign(tf.div(asset_weights, result_sum)) //If sum > 1, dividing it by the sum scales it back to 1

    const returns_sum = tf.mul(asset_weights, assetRets).sum();
    // returns_sum.print();
};
```

Now we start the optimisation process.
```
// The main training function
async function train(epochs, done){
    for (let i=0; i<epochs; i++){
        let cost;

        cost  = tf.tidy(()=>{
            cost = optimizer.minimize(()=>{
                const constraining = constraints();
                const sharpe = predict();
                
                return sharpe;
            },true);
            const constraining = constraints();
            return cost;
        })
        // If we use this line below, we will be pushing in a new point for each iteration
        // cost.data().then((data)=>lossArray.push({i:i, error:-data[0]}));

        progresspercent = 100*i/epochs;
        $('.progress-bar').css('width', progresspercent+'%').attr('aria-valuenow', progresspercent);

        if(i%100==0){
            // await cost.data().then((data)=>console.log(i,data));
            cost.data().then((data)=>lossArray.push({i:i, error:-data[0]}));
            // console.log('Run:', i);
            // cost.print();
            // asset_weights.print();
        }
        await tf.nextFrame();
    }

    // We call the functions in done() later followed by the plotting functions later
    done();

    // Important that we call this here with await to ensure that the operations above are finished first
    // Placing it in the done() function which is not an async function does not help!
    await renderPie(asset_weights_array);
    await ploterrors(lossArray);
    await plot_optimal(finalrets/100,finalvols/100);
}
```

And we start the training/optimisation process with the **Optimise Portfolio** button. That whole mess of code after the ‘Training Completed’ line basically runs some computations only when training is completed, and prints the final portfolio return and volatility to the webpage.
```
$('#optimise').click(()=>{

    update();
    train(epochs, ()=>{
        // console.log('Training Completed');

        asset_weights_array = [];

        let final_asset_weights= asset_weights.dataSync();
        // console.log(final_asset_weights);

        for(let i=0; i<asset_names.length; i++){
            asset_weights_array.push({Asset:asset_names[i], Proportion:final_asset_weights[i]});
        }
        // console.log('After:', asset_weights_array);


        // Write the final return and vol of the portfolio to the screen 
        weighted_vols = asset_weights.mul(assetVols);
        vol = weighted_vols.transpose().matMul(corrs).matMul(weighted_vols).squeeze();
        finalvols = 100*Math.sqrt(vol.dataSync());
        $('#pfvol').text(finalvols.toFixed(2)+'%');

        returns_sum = asset_weights.mul(assetRets).sum();
        finalrets = 100*returns_sum.dataSync();
        $('#pfreturn').text(finalrets.toFixed(2)+'%');
        sharpe = tf.neg(tf.div(returns_sum,vol));

        

        // This will get called before the operations above are completed
        // render(asset_weights_array);
    });
    
});
```

That’s kind of it. I shall not go into the D3.js code used to draw the charts as they are pretty much the same as what I have covered before in my [3 Days of Hand Coding Visualisations][4] post.

The full code can be found [here][5].

This code here is already released under the MIT License (i.e. it is provided as is, without any warranty), but just to be safe, I am going to state that this should not be relied upon for any investment decision!




[1]:	https://medium.com/quaintitative/data-exploration-in-pandas-f7cd1a3b3594
[2]:	https://medium.com/quaintitative/quickstart-to-visualising-and-analysing-financial-data-with-pandas-bbd835c9c560
[3]:	https://playgrdstar.github.io/portfolio_optimisation_with_tensorflow/
[4]:	https://medium.com/creative-coding-space/3-days-of-hand-coding-visualisations-introduction-64da30d8793f
[5]:	https://github.com/playgrdstar/portfolio_optimisation_with_tensorflow