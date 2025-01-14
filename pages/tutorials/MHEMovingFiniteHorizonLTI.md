---
layout: single
title: Moving finite-horizon MHE filter tutorial for LTI systems
showTitle: true
excerpt: "Tutorial on decentralized MHE filter synthesis using the moving finite-horizon method."
permalink: /tutorials/MHEMovingFiniteHorizonLTI/
classes: wide
sidebar:
  title: "Tutorials"
  nav: sidebar-tutorials
header:
  teaser: "/assets/img/MHEMovingFiniteHorizonLTITutorial_trace.svg"
tags:
  - tutorial
  - moving-finite-horizon
  - lti
  - estimation
  - moving-horizon-estimation
date: "2022-04-05"
last_modified_at: "2022-04-05"
---
{{page.excerpt}}\\
See [documentation for MHEMovingFiniteHorizonLTI](/documentation/MHEMovingFiniteHorizonLTI/) for more information.

***

To open this tutorial execute the following command in the MATLAB command window
~~~m
open MHEMovingFiniteHorizonLTITutorial
~~~

Use synthetic system matrices $\mathbf{A}$, $\mathbf{C}$, $\mathbf{Q}$, $\mathbf{R}$, and $\mathbf{E}$ from [[Section 4.1, 1]](#references)
~~~m
n = 5;
o = 4;
A = [0.152  0.092   0.235   0.642   0.506;
     0.397  0.615   0.448   0.221   0.279;
     0.375  0.011   0.569   0.837   0.747;
     0.131  0.573   0.061   0.971   0.237;
     0.435  0.790   0.496   0.846   0.957];
C = [0.620  0.255   0.725   0.404   0.511;
     0.600  0.859   0.230   1.988   0.061;
     0.173  0.911   0.576   0.090   0.726;
     0.090  0.700   0.811   0.321   0.557];
Q = [3.318  4.662   1.598   -1.542  -1.999;
     4.662  11.520  2.608   -2.093  -5.442;
     1.598  2.608   4.691   0.647   -0.410;
     -1.542 -2.093  0.647   2.968   0.803;
     -1.999 -5.442  -0.410  0.803   2.851];
R = [3.624  2.601   -0.042  -0.944;
     2.601  7.343   -0.729  -2.786;
     -0.042 -0.729  0.745   -0.242;
     -0.944 -2.786  -0.242  1.612];
E = [1   0   1   1;
     0   1   0   1;
     0   0   1   0;
     1   1   1   0;
     1   1   0   1];
~~~

Synthesize decentralized **moving horizon estimation (MHE)** sequence of gains using the **moving finite-horizon** method (with some optional parameters) with **null initial covariance**
{: .text-justify}
~~~m
opts.verbose = true;
opts.maxIt = 100;
opts.P0 = zeros(size(A));
opts.epsl_inf = 1e-4;
W = 5;
[Kinf,Pinf] = MHEMovingFiniteHorizonLTI(A,C,Q,R,E,W,opts);
Kinf
trace(Pinf)
~~~
~~~text
----------------------------------------------------------------------------------
Running moving finite horizon algorithm with:
epsl_inf = 0.0001 | W = 5 | maxIt = 100 .
Convergence reached with: epsl_inf = 0.0001 | W = 5 | maxOLIt = 100
A total of 8 outer loop iterations were run.
---------------------------------------------------------------------------------
Kinf =
  5×1 cell array
    {5×4 double}
    {5×4 double}
    {5×4 double}
    {5×4 double}
    {5×4 double}

ans =
   17.8864
~~~

Now choose the best performing sequence among the gain sequence synthesis procedures with **1000 randomly generated initial covariance** matrices
{: .text-justify}

~~~m
noP0 = 500;
opts.verbose = false;
opts.maxIt = 100;
opts.epsl_inf = 1e-4;
W = 5;

% Synthezize several sequences
Pinf = inf(size(A));
rng(0); % For consistency
for i = 1:noP0
    aux = 2*(rand(size(A))-0.5);
    opts.P0 = aux*aux';
    [auxK,auxP] = MHEMovingFiniteHorizonLTI(A,C,Q,R,E,W,opts);
    if i == 1 || trace(auxP) < trace(Pinf)
        Pinf = auxP;
        Kinf = auxK;
    end
end

Kinf
trace(Pinf)
~~~
~~~text
Kinf =
  5×1 cell array
    {5×4 double}
    {5×4 double}
    {5×4 double}
    {5×4 double}
    {5×4 double}

ans =
   17.8804
~~~

Notice that **slightly better steady-state performance** was achieved, as expected.


Simulate the error dynamics for the synthetic system
{: .text-justify}
~~~m
% Simulation span
T = 100;
% Initial estimation erro covariance matrix
P0 = 100*eye(n);
% Initialise error cell
x = cell(T,1);
P = cell(T,1);
% Generate random initial error
x0 = transpose(mvnrnd(zeros(n,1),P0));
% Initialize vectors to hold the process and sensor noise
% (for the simulation of estimation error dynamics)
w0_noise = mvnrnd(zeros(n,1),Q)';
w_noise = cell(T,1); % process noise (the T-th entry is unused)
v_noise = cell(T,1); % sensor noise
% Synthesize OS gains for initialization while k<W
[KOS,POS] = kalmanOneStepLTI(A,C,Q,R,E);

for k = 1:T  
    %%%%% Estimation error dynamics
    % Random process and sensor Gaussian noise
    w_noise{k,1} = mvnrnd(zeros(n,1),Q)';
    v_noise{k,1} = mvnrnd(zeros(o,1),R)';

    %%%%% Simulation
    if k-W >= 1
        P_aux = P{k-W,1};
        x_aux = x{k-W,1};
        for j = 1:W
            P_aux = A*P_aux*A'+Q;
            P_aux = Kinf{j,1}*R*Kinf{j,1}'+(eye(n)-Kinf{j,1}*C)*P_aux*(eye(n)-Kinf{j,1}*C)';           
            x_aux = (eye(n)-Kinf{j,1}*C)*(A*x_aux+w_noise{k-W+j-1,1})-Kinf{j,1}*v_noise{k-W+j,1};
        end
        x{k,1} = x_aux;
        P{k,1} = P_aux;
    else % One-step to initialize
        if k == 1
            P_aux = P0; % Covariance prediction
            x_aux = x0; % Get estimate at the beggining of the window
            x{k,1} = (eye(n)-KOS*C)*(A*x_aux+w0_noise)-KOS*v_noise{k,1};
        else
            P_aux = P{k-1,1};
            x_aux = x{k-1,1};
            x{k,1} = (eye(n)-KOS*C)*(A*x_aux+w_noise{k-1,1})-KOS*v_noise{k,1};
        end     
        P_aux = A*P_aux*A'+Q;
        P{k,1} = KOS*R*KOS'+(eye(n)-KOS*C)*P_aux*(eye(n)-KOS*C)';
    end
end
~~~
Plot the evolution of the **norm of the estimation error** and of the **trace of the covariance** matrix
~~~m
% Plot the ||x||_2 vs instant of the simulation
figure;
hold on;
set(gca,'FontSize',35);
ax = gca;
ax.XGrid = 'on';
ax.YGrid = 'on';
xPlot = zeros(T+1,1);
xPlot(1,1) = norm(x0);
for j = 1:T
    xPlot(j+1,1) =norm(x{j,1}(:,1));
end
plot(0:T, xPlot(:,1),'LineWidth',3);
set(gcf, 'Position', [100 100 900 550]);
ylabel('$\|\mathbf{x}_{MFH}(k)\|_2$','Interpreter','latex');
xlabel('$k$','Interpreter','latex');
hold off;

% Plot the covariance trace
figure;
hold on;
set(gca,'FontSize',35);
ax = gca;
ax.XGrid = 'on';
ax.YGrid = 'on';
trPlot = zeros(T+1,1);
trPlot(1,1) = trace(P0);
for j = 1:T
    trPlot(j+1,1) =trace(P{j,1});
end
plot(0:T, trPlot,'LineWidth',3);
set(gcf, 'Position', [100 100 900 550]);
ylabel('$\mathrm{tr}(\mathbf{P}_{MFH}(k|k))$','Interpreter','latex');
xlabel('$k$','Interpreter','latex');
hold off;
~~~

{:refdef: style="text-align: center;"}
![image-title-here](/assets/img/MHEMovingFiniteHorizonLTITutorial_trace.svg){:class="img-responsive"}
{: refdef}

{:refdef: style="text-align: center;"}
![image-title-here](/assets/img/MHEMovingFiniteHorizonLTITutorial_error.svg){:class="img-responsive"}
{: refdef}


{: .text-justify}

# References
[1] {% for paper in site.data.references%}{% if paper.key == "Viegas2018Discrete" %}
<a href="{{paper.url}}" target="_blank">{{paper.harvardCitation}}</a>
{: .text-justify}
{% endif %}{% endfor %}
