# Google's Page Rank Algorithm in T SQL

We are given the following schemea:

nodes(paperID, paperTitle)

edges(paperID, citedPaperID)

~~~~sql
-- drop/create helper tables
DROP TABLE IF EXISTS NumOut;
DROP TABLE IF EXISTS CurrentRank;
DROP TABLE IF EXISTS NextRank;
DROP TABLE IF EXISTS Sinks;
DROP TABLE IF EXISTS NewEdges;

CREATE TABLE NumOut(id int PRIMARY KEY, degree int);
CREATE TABLE CurrentRank(id int PRIMARY KEY, rank float);
CREATE TABLE NextRank(id int PRIMARY KEY, rank float);
CREATE TABLE Sinks(id int PRIMARY KEY);
CREATE TABLE NewEdges(paperID int, citedPaperID int);

-- delete all records
DELETE FROM NumOut;
DELETE FROM CurrentRank;
DELETE FROM NextRank;
DELETE FROM SINKS;

-- initialize variables
DECLARE @d FLOAT = 0.85;
DECLARE @Condition FLOAT;
DECLARE @n INTEGER;

-- total number of nodes
SELECT @n = COUNT(*) FROM nodes;

-- insert edges into new edges
INSERT INTO NewEdges
SELECT * FROM edges;

-- find sinks
INSERT INTO Sinks
SELECT nodes.paperID
FROM nodes 
WHERE paperID not in (SELECT paperID from edges);

-- insert edges from sinks to all nodes into new edges
-- this simulates "sinks link w = prob to everyone else"
INSERT INTO NewEdges
SELECT sinks.id, nodes.paperID
FROM Sinks CROSS JOIN nodes
WHERE sinks.id != nodes.paperID;

--create table that keeps track of
--how many citations out of each paper
INSERT INTO NumOut
SELECT nodes.paperID, COUNT(*)
FROM nodes INNER JOIN NewEdges
ON nodes.paperID = NewEdges.paperID
GROUP BY nodes.paperID;

--create table to keep track of current page rank
--initialize as 1/n for each node
INSERT INTO CurrentRank
SELECT nodes.paperID, 1.0/@n AS Rank
FROM nodes INNER JOIN NumOut
ON nodes.paperID = NumOut.id;

SET @Condition = 1;

WHILE @Condition >= 0.01
BEGIN
	-- Calculate next page rank iteration
	DELETE FROM NextRank;
	INSERT INTO NextRank
	SELECT e.citedPaperID, ((1.0-@d)/@n) + @d * SUM(IndivRank)
	FROM (
		SELECT NewEdges.citedPaperID, 1.0*CurrentRank.rank/NumOut.degree AS IndivRank
		FROM CurrentRank LEFT OUTER JOIN NewEdges ON CurrentRank.id = NewEdges.paperID
				         LEFT OUTER JOIN NumOut ON CurrentRank.id = NumOut.id) e
	GROUP BY e.citedPaperID;

	--calculate stopping condition
	SELECT @Condition = SUM(ABS(CurrentRank.Rank - NextRank.Rank))
	FROM CurrentRank INNER JOIN NextRank ON CurrentRank.id = NextRank.id;

	--replace next rank with current page rank
    DELETE FROM CurrentRank;
    INSERT INTO CurrentRank
    SELECT * FROM NextRank;
	
END

-- print solution
SELECT TOP(10) c.id, n.paperTitle ,c.rank
FROM CurrentRank c INNER JOIN nodes n ON c.id = n.paperID
ORDER BY rank DESC;

GO

~~~~

# Result

The 10 pages with the highest Page Rank Probability:

| id      | paperTitle                                                          | rank                |
|---------|---------------------------------------------------------------------|---------------------|
| 9504090 | Massless Black Holes and Conifolds in String Theory                 | 0.0147242485713874  |
| 9510135 | Bound States Of Strings And p-Branes                                | 0.0144463053478479  |
| 9711200 | The Large N Limit of Superconformal Field Theories and Supergravity | 0.0136475828170161  |
| 9802150 | Anti De Sitter Space And Holography                                 | 0.00969790725741225 |
| 208020  | Open strings and their symmetry groups                              | 0.00862989546971674 |
| 9602065 | D--branes and Spinning Black Holes                                  | 0.00771630107059198 |
| 9305185 | Duality Symmetries of 4D Heterotic Strings                          | 0.00754976747481236 |
| 9611050 | TASI Lectures on D-Branes                                           | 0.00712937876530993 |
| 9501030 | Strong/Weak Coupling Duality from the Dual String                   | 0.00581545481823665 |
| 9602135 | Entropy and Temperature of Black 3-Branes                           | 0.00541617199831115 |
