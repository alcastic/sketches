# Fan-out Fan-in

## Abstract

This docuements aims to introduce the **Finin-Faout** pattern for a concurrent data processing pipelines and contrasting against a traditional secuential not concurrent design. Examples will be written in *Go Programing Languaje* given its natural nature and abstractions for bilding concurrent algorithms.

## Pipelines - Data Procesing 

A pipeline is a chain/serie of data processing elements, commonly known as *stages* which:

1. Take data in (inputs).
2. Perform an operation on the data (data processing).
3. Pass the data out (outputs).

Pipeline allows us to split our data processing flow in smaller and easy to test and maintain logical unit. Using pipelines in our flows provide the benefits of separation of cencenr, divide and conquer strategy and get after single-responsibility principle.

## Secuential Pipeline Data processing

Lets see a simple (but complete) sample of data processing.