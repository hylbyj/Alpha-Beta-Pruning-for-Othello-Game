PART1:
1) A minimax Othello player algorithm:
heuristics of minimax Othello player: The heuristic is rather simple for minimax algorithm because we don’t consider time as an important requirement (we don’t use it to compete with other agents.) So we can simply increase our winning chances by increasing the depth. My heuristic function is simply a combination of greedy algorithm and corner capture strategy. I assigned them with different weights which can produce the better performance. The weight matrix is extract from the paper called “An Analysis of Heuristic in Othello”. The weight matrix used in the algorithm is:
    WEIGHTS = [4, -3, 2, 2, 2, 2, -3, 4,
               -3, -4, -1, -1, -1, -1, -4, -3,
               2, -1, 1, 0, 0, 1, -1, 2,
               2, -1, 0, 1, 1, 0, -1, 2,
               2, -1, 0, 1, 1, 0, -1, 2,
               2, -1, 1, 0, 0, 1, -1, 2,
               -3, -4, -1, -1, -1, -1, -4, -3,
               4, -3, 2, 2, 2, 2, -3, 4]
The heuristic value of a player is calculated by adding together the weights of the squares where player is captured. The difference between two players is also the key to winning the game. I add them together in the heuristic function. In order to get better performance I set the depth as 5. Based on my observation, larger depth does not necessarily guarantee winning result since as for random opponent, we can never predict what they would play in the next round.
For the time management issue, I set a limitation of remaining time as 10 seconds. When there is less than 7 seconds left in the game, the agent would implement greedy algorithm. This also reasonable. Because my heuristic function concentrated more on corner capture which would be the most effective at the beginning of the game.
Three functions that are related to heuristics function:
    def heuristic(self, board, color):
        return  self.cornerweight(color, board) + self._get_cost(board, color)

    def cornerweight(self, color, board):
        total = 0;
        i = 0;
        while i < 64:
            if board[i/8][i%8] == color:
               total += StudentEngine.WEIGHTS[i];
               #print "weights" + str(i) + "number:"+ str(StudentEngine.WEIGHTS[i])
            if board[i/8][i%8] == -color:
               total -= StudentEngine.WEIGHTS[i];
               #print "weights" + str(i) + "number:"+ str(StudentEngine.WEIGHTS[i])
            i += 1
        print "cornerweight" + str(total)
        return total
    def _get_cost(self, board, color):
        """ Return the difference in number of pieces after the given move
        is executed. """

        # Create a deepcopy of the board to preserve the state of the actual board
        #newboard = deepcopy(board)
        #newboard.execute_move(move, color)

        # Count the # of pieces of each color on the board
        num_pieces_op = board.count(-color)
        num_pieces_me = board.count(color)
        print "_get_cost" + str(num_pieces_me - num_pieces_op)
        # Return the difference in number of pieces
        return num_pieces_me - num_pieces_op


How I implement minimax algorithm:
I separated minimax into three functions, the first one is minimax which is the general function acting like get the min or the max value based on the depth it currently explores:
    def _minmax(self, board, color, move_num, time_remaining, time_opponent, ply):
        #need to get all the legal moves
        #def value(board):
        #    return self.minmax(board, -color, move_num, time_remaining, time_opponent, ply-1)[0]
        #if ply == 0:
        #   return board.count(color), None
        moves = board.get_legal_moves(color)
        #if time_remaining < 5:
        #   return (0, max(moves, key=lambda move: self.greedy(board, color, move)) )

        print "leagal move" + str(moves)
        if not isinstance(moves, list):
           score = self.heuristic(board, color)
           return score,None
        if time_remaining < 10:
           return (0, max(moves, key=lambda move: self.greedy(board, color, move)) )
        print ply
        return_move = moves[0]
        bestscore = - StudentEngine.INFINITY
        print "using minmax best score:"+ str(bestscore)
        #ply = 4
        #will define ply later;
        for move in moves:
            newboard = deepcopy(board)
            newboard.execute_move(move,color)

            score = self.min_score(newboard, -color, move_num, ply)
            if score > bestscore:
                bestscore = score
                return_move = move
                print "return move" + str(return_move) + "bestscore" + str(bestscore)
        #newboard = deepcopy(board)
        #return max((value(newboard.execute_move(m, color)),m) for m in moves)
        return (bestscore,return_move)
function max_score is aimed to maximize the score of player and function min_score is aimed to minimize the score of opponent. They all only return the value.
    def max_score(self, board, color, move_num, ply):
        #print "move_num" + str(move_num)
        moves = board.get_legal_moves(color)
        #if not isinstance(moves, list):
        #   return board.count(color)
        if ply == 0:
           return self.heuristic(board, color)
        bestscore = -StudentEngine.INFINITY
        for move in moves:
            newboard = deepcopy(board)
            newboard.execute_move(move,color)
            score = self.min_score(newboard, -color, move_num, ply-1)
            if score > bestscore:
                bestscore = score
        return bestscore

    def min_score(self, board, color, move_num, ply):
        #print "move_num" + str(move_num)
        moves = board.get_legal_moves(color)
        #if not isinstance(moves, list):
        #   return board.count(color)
        if ply == 0:
           return self.heuristic(board, color)
        bestscore = StudentEngine.INFINITY
        for move in moves:
            newboard = deepcopy(board)
            newboard.execute_move(move,color)
            score = self.max_score(newboard, -color, move_num, ply-1)
            if score < bestscore:
                bestscore = score
        return bestscore
Results of running my agent versus random:
FINAL BOARD
--

    A B C D E F G H
    ---------------
8 | B B B B B B B B | 8
7 | W B B W W B B B | 7
6 | W W W W B B B B | 6
5 | W W B B B B B B | 5
4 | B B W W B B B B | 4
3 | B B B W B B B B | 3
2 | B W B W W W B B | 2
1 | B B B B B B B B | 1
    ---------------
    A B C D E F G H
2) Alpha-beta Othello player
I modify the three functions and initially set alpha, beta as +infinity and -infinity. The functions are listed:
    def _minmax_with_alpha_beta(self, board, color, move_num, time_remaining, time_opponent, ply):
        moves = board.get_legal_moves(color)
        #print "leagal move" + str(moves)
        if not isinstance(moves, list):
           score = board.count(color)
           return score, None

        #print ply
        return_move = moves[0]
        bestscore = - StudentEngine.INFINITY
        #print "using alpha_beta best score:"+ str(bestscore)
        #ply = 4
        #will define ply later;
        for move in moves:
            newboard = deepcopy(board)
            newboard.execute_move(move,color)

            score = self.min_score_alpha_beta(newboard, -color, move_num, ply, StudentEngine.INFINITY, -StudentEngine.INFINITY)
            if score > bestscore:
               bestscore = score
               return_move = move
               #print "return move" + str(return_move) + "best score" + str(bestscore)

        return (bestscore,return_move)

Also the max and min value function:
    def max_score_alpha_beta(self, board, color, move_num, ply, alpha, beta):
        if ply == 0:
            return self.heuristic(board, color)
        bestscore = -StudentEngine.INFINITY
        for move in board.get_legal_moves(color):
            newboard = deepcopy(board)
            newboard.execute_move(move,color)
            score = self.min_score_alpha_beta(newboard, -color, move_num, ply-1, alpha, beta)
            if score > bestscore:
                bestscore = score
            if bestscore >= beta:
                return bestscore
            alpha = max (alpha,bestscore)
        return bestscore

    def min_score_alpha_beta(self, board, color, move_num, ply, alpha, beta):
          if ply == 0:
             return self.heuristic(board, color)
          bestscore = StudentEngine.INFINITY
          for move in board.get_legal_moves(color):
              newboard = deepcopy(board)
              newboard.execute_move(move,color)
              score = self.max_score_alpha_beta(newboard, -color, move_num, ply-1, alpha, beta)
              if score < bestscore:
                 bestscore = score
              if bestscore <= alpha:
                 return bestscore
              beta = min(beta,bestscore)
          return bestscore
Some of the results:
FINAL BOARD
    A B C D E F G H
    ---------------
8 | W W W W W W W W | 8
7 | W W W W W B W W | 7
6 | W B W B B W B W | 6
5 | W B W B W W B W | 5
4 | W B W W W B B W | 4
3 | W B W W W W B W | 3
2 | W W W W B B B W | 2
1 | W W W W W W W W | 1
    ---------------
STATISTICS (score / remaining time):
Black: 16 / 30.0
White: 48 / 28.8

- yh2617 (white) wins the game! (48-16)



