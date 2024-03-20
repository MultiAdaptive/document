# KZG Polynomial Commitments

This is a brief introduction to the polynomial commitment scheme KZG, not a strictly mathematical or cryptographic description.



## Aggregated KZG

Aggregated KZG, when used in Data Audit, is a sophisticated cryptographic technique designed to make blockchain networks more efficient and scalable. To explain this in an easier way, let's break it down into simpler concepts:

### What is KZG?

KZG stands for Kate-Zaverucha-Goldberg, named after the researchers who developed it. It's a method for creating a compact cryptographic "commitment" to a set of data (like a list of transactions in a blockchain block) without revealing the data itself. Think of it as a way to securely lock data in a box and then prove what's inside the box without opening it.

### What Does "Aggregated" Mean?

"Aggregated" refers to the process of combining multiple things into a single one. In the context of KZG, it means taking multiple cryptographic commitments and combining them into one. This is like taking several locked boxes and merging them into a single box that still securely contains all the original items.

### How Does This Apply to Data Audit?

Data Audit is a technique used in blockchain networks to ensure that the extremely large data is completely stored in the server and has not been lost or tampered with. It's like if you submit a library of books to the cloud, and you can check the integrality of all the books without download it to your laptop.

### Why Use Aggregated KZG in Data Audit?

1. **Efficiency**: By aggregating KZG commitments, the process of verifying data availability becomes much more efficient. Instead of checking each piece of data individually, validators in the network can check a single aggregated commitment. This is akin to verifying the contents of a single box rather than opening and checking multiple boxes one by one.

2. **Scalability**: This efficiency directly contributes to scalability. Blockchain networks can handle more transactions and operate faster because the data availability verification process is streamlined. It's like being able to quickly check that all the books in the library are in place without having to look at each book individually.

3. **Security**: Aggregated KZG commitments maintain the security properties of the individual commitments. Even though the commitments are combined, the data is still securely "locked" and can be verified without being exposed. This ensures that the network remains secure and tamper-proof.

In summary, using aggregated KZG in Data Audit is like having a super-efficient, secure library system. It ensures that all the books (data) are available when needed, without everyone having to hold every book. This makes the blockchain network faster, more scalable, and secure, enabling it to support more users and transactions.

## Introduction

KZG polynomial commitment, KZG for short, is a scheme that allows a prover to prove the value of a polynomial at any position without giving out the polynomial itself. 

It is like this:

**Prover** :  I have a polynomial $f(x)$, and its commitment is $cm_f$ , now send the $cm_f$ to you.

**Verifier** : Thanks, I get the $cm_f$， can you help to compute the value of the polynomial at the position $a$?

**Prover** : Yes, let me compute $b=f(a)$, and generate a proof $p_a$, and send $b,p_a$ to you.

**Verifier** : Get it! I have checked your proof $p_a$ with the commitment $cm_f$, and now I know that you are honest, and I can believe that $b=f(a)$, even though I don't know what $f(x)$​ is. 



## Polynomial

If you have a set of number $(a_0,a_1,...,a_n)$, then you can get a polynomial $f(x):$

$$
f(x)=a_0,a_1x+a_2x^2+...a_nx^n
$$

and the degree of $f(x)$ is $n$.

For example, we have (3, 2), then we get $f(x)=3+2x$, and its degree is 1, we can draw it like this:

<img src="images/line.png" alt="image-20240313110622731" style="zoom:25%;" />

it is a line. 

If we have (1, 2, 1), then we get $f(x)=1+2x+x^2$, and its degree is 2, we can draw it like this:

<img src="images/parabola.png" alt="image-20240313111430001" style="zoom: 67%;" />

it is a parabola.

We can draw any polynomial on the plane as a curve.

As you may have heard, two points determine a line, and three points determine a parabola. Yes, if we have $n+1$ points, then we can determine a polynomial with degree $n:$

$$
(a_0,f(a_0)),(a_1,f(a_1)),...(a_n,f(a_n))\Longrightarrow f(x)=b_0+b_1x+...+b_nx^n
$$

If we have two polynomial $f(x)=a_0+a_1x+...a_nx^n$ ,  $p(x)=b_0+b_1x+...+b_nx^n$, and a number $c$, then we can have $:$

$$
g(x)=f(x)+p(x)=(a_0+b_0)+(a_1+b_1)x+...+(a_n+b_n)x^n
$$

$$
t(x)=cf(x)=ca_0+ca_1x+...ca_nx^n
$$

Yes, we can perform any linear combination of polynomials, and it is still a polynomial.

## Data Audit

If you store a piece of data in a server, is there any way to ensure that the data in the server is properly saved and not lost? The simplest way is to ask the server to send all the data back and you can check it again. But this method is too inefficient. Is there an easy way to accomplish this task? In fact, KZG can achieve it very well.

First, you can encode your data to a polynomial $f(x)$, and compute its KZG commitment $cm_f$. You publish the $cm_f$ (send it to Ethereum for example) and send $f(x)$ to polynomial to the server.

Next, you are the **Verifier**, and the server is the **Prover** :

**Verifier** : Can you help to comupte the value of polynomial $f(x)$ at the position $a$ ? If you hold the whole polynomial, it is an easy task. Don't forget to submit your proof.

**Prover** : Yes, let me compute $b=f(a)$, and generate a proof $p_a$, and send $b,p_a$ to you.

**Verifier** : Get it! I have checked your proof $p_a$ with the commitment $cm_f$, and now I know that you are honest, and didn't lose my data.

How about if we store some piece of data so that we have a set of polynomial $f_i(x)$? We can use **Aggregate KZG** to audit all the data at once.

At the same, we have the KZG commitment set $cm_{f_i}$ for $f_i(x)$. KZG has a linear feature: the commitment of the polynomial $h(x)=\sum k_if_i(x)$ is $cm_h=\sum k_icm_{f_i}$.

**Verifier** : I have a set $\{k_i\}$, can you help to comupte the value of polynomial $h(x)=\sum k_if_i(x)$ at the position $a$ ? If you hold the whole polynomials, it is an easy task. Don't forget to submit your proof.

**Prover** : Yes, let me first comupte $h(x)=\sum k_if_i(x)$ , then compute $b=h(a)$, and generate a proof $p_a$, and send $b,p_a$ to you.

**Verifier**: Get it! I have checked your proof $p_a$ with the commitment $cm_h=\sum k_icm_{f_i}$, and now I know that you are honest, and didn't lose my data.



## Data Availability Sampling.

If the data is so large that the network transmission fails, then we need to sample the data.

For example, the server stored the data as a polynomial $f(x)=a_0+a_1x+...+a_{100}x^{100}$ with degree 100. If we want to sample 1/10 of the whole data, we can ask for 10 random points of $f(x):$

$$
(b_0,f(b_0)),(b_1,f(b_1)),...,(b_9,f(b_9))
$$


Don't forget to ask for a KZG proof at the same time to make sure that the server is honest.

If we have 10 shardings of the whole data, then we have 100 points of $f(x)$, then we can rebuild it $:$

$$
(b_{0,0},f(b_{0,0})),(b_{0,1},f(b_{0,1})),...,(b_{0,9},f(b_{0,9}))
$$

$$
(b_{1,0},f(b_{1,0})),(b_{1,1},f(b_{1,1})),...,(b_{1,9},f(b_{1,9}))
$$

$$
...
$$
