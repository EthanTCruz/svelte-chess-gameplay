<script>

	import { board } from './store.js'	

	let turn = 'W';
	let move_no;
	let positions_selected = 0;
	let move_from;
	let move_to;
	let last_move = [0,0];
	let en_passant = [0,0];

	let b_castle = true
	let w_castle = true
	let w_king_has_moved = false
	let b_king_has_moved = false
	let w_rook_has_moved = [false,false]
	let b_rook_has_moved = [false,false]
	function checkMove(){
		let piece = board[move_from[1]].positions[move_from[0]]
		
		let team = piece.charAt(piece.length - 1);
		if (team != turn){
			return false
		}
		piece = piece.charAt(0);

		if (piece == 'P'){
			//add check en passant
			let pawnCheck = checkPawn(piece,team)
			if (pawnCheck){
				if (turn == 'W'){
					if (move_to[1] == 7){
						let pieceChoice = prompt("Please enter Q, N or R")
						while (!(pieceChoice == 'Q'||pieceChoice=='N'||pieceChoice=='R')){
						 pieceChoice = prompt("Please enter Q, N or R")
						}
						board[move_from[1]].positions[move_from[0]] = `${pieceChoice}W`;

					}
				} else {
					if (move_to[1] == 0){
						let pieceChoice = prompt("Please enter Q, N or R")
						while (!(pieceChoice == 'Q'||pieceChoice=='N'||pieceChoice=='R')){
						 pieceChoice = prompt("Please enter Q, N or R")
						}
						board[move_from[1]].positions[move_from[0]] = `${pieceChoice}B`;

					}
				}
			}

			return pawnCheck
		} else if (piece == 'N'){
			return checkKnight(piece,team)
		} else if (piece == 'R') {
			return checkLaterally(team)
		} else if (piece == 'B'){
			return checkDiagonally(team)
		} else if (piece == 'Q'){
			return checkDiagonally(team) || checkLaterally(team)
		} else if (piece == 'K'){
			//add check castle
			
			return checkKing(team)
		}
		console.log(`b check: ${in_check('B')}`)
		console.log(`w check: ${in_check('W')}`)

		return true || in_check(team)
	}

	function checkPawn(piece,team){
		let direction;
		let opponent;
		let row_check = move_from[1] >= 1 && move_from[1] <= 6
		let cond1
		let cond2
		let cond3
		let cond4
		let cond5 = false
		let right_check
		let left_check 
		if (team == 'W'){
			direction = 1
			opponent = 'B'
			cond1 = move_from[1]+2*direction == move_to[1] && move_from[1] == 1
		} else {
			direction = -1
			opponent = 'W'
			cond1 = move_from[1]+2*direction == move_to[1] && move_from[1] == 6

		}
		if (cond1){
			en_passant[0] = move_to[0]
			en_passant[1] = move_to[1]-1*direction

		}

			if (en_passant[0] == move_to[0] && en_passant[1] == move_to[1]){
				cond5 = true
			}

		
		right_check = 1 <= move_from[0]+1 && move_from[0]+1 <= 7
		right_check = right_check && board[move_to[1]].positions[move_to[0]].charAt(1) == opponent
		left_check = 2 <= move_from[0]+1 && move_from[0]+1 <= 8
		left_check = left_check && board[move_to[1]].positions[move_to[0]].charAt(1) == opponent
		
		cond2 = move_from[1]+1*direction == move_to[1] && move_from[0] == move_to[0] 

		if (right_check){
			cond3 = move_from[1]+1*direction == move_to[1] && move_from[0]+1 == move_to[0]

		} else {
			cond3 = false
		}
		if (left_check){
			cond4 = move_from[1]+1*direction == move_to[1] && move_from[0]-1 == move_to[0]

		} else {
			cond4 = false
		}

		return (cond1 || cond2 || cond3 || cond4 || cond5)


	
	}

	function checkKing(team){
		let x = move_from[0] - move_to[0]
		let y = move_from[1] - move_to[1]
		x = x*x
		y=y*y
		let opponent

		if ((x == 0 || x == 1)&&(y == 0||y == 1)&&(board[move_to[1]].positions[move_to[0]].charAt(1) != team)){
			return true
		}
		return false
	}

	function checkDiagonally(team){
		let x = move_from[0] - move_to[0]
		let y = move_from[1] - move_to[1]
		let direction = [1,1]

		if (x > 0){
			direction[0] = -1
		}
		if (y > 0){
			direction[1] = -1
		} 
		return checkDiagonal(team,direction,move_from)	
	}

	function checkDiagonal(team,direction,location){
		let opponent

		if (team == 'W') {
			opponent = 'B'
		} else {
			opponent = 'W'
		}




		let next = [0,0]
		next[0] = location[0]
		next[1] = location[1]
		next[0] = next[0] + direction[0]
		next[1] = next[1] + direction[1]
		if (next[0] == 8 || next[0] == -1 || next[1] == 8 || next[1] == -1)
		return false

		
		
		let next_space_team = board[next[1]].positions[next[0]].charAt(1)
		if (next_space_team == opponent){
			if (next[0] == move_to[0] && next[1] == move_to[1]){
				return true
			}
			return false
		} else if (next_space_team == ' '){
			if (next[0] == move_to[0] && next[1] == move_to[1]){
				return true
			}
			return checkDiagonal(team=team,direction=direction,next=next)
		} else { 
			return false
		}


	}

	function checkLaterally(team){
		let x = move_from[0] - move_to[0]
		let y = move_from[1] - move_to[1]
		let direction

		if (x == 0){
		if (y < 0){
			direction = 1
		} else if (y > 0){
			direction = -1
		} else {
			return false
		}
		let check = checkDirection(team,direction,1,move_from)
		if (getPiece(move_from[0],move_from[1]).charAt(0) == 'R' && check){
			if (team == 'W'){
				if (move_from[0] == 0 && move_from[1]==0){
						w_rook_has_moved[0] = true
				} else if (move_from[0] == 0 && move_from[1] == 7){
						w_rook_has_moved[1] = true
				}

			} else 	if (team == 'B'){
				if (move_from[0] == 7 && move_from[1]==0){
						b_rook_has_moved[0] = true
				} else if (move_from[0] == 7 && move_from[1] == 7){
						b_rook_has_moved[1] = true
				}

			}
		}

		return checkDirection(team,direction,1,move_from)	
		} else 	if (y == 0){		
		if (x < 0){
			direction = 1
		} else if (x > 0){
			direction = -1
		} else {
			return false
		}

			return checkDirection(team,direction,0,move_from)	
		} else {
			return false
		}
			
	}
function in_check_location(team,location){
	let checks = InCheckByDiagonally(team,location)
		console.log(`check1: ${checks}`)
		checks = checks || InCheckByLaterally(team,location)
		console.log(`check2: ${checks}`)
		checks = checks || InCheckByKing(team,location)
		console.log(`check3: ${checks}`)
		checks = checks || InCheckByKnight(team,location)
		console.log(`check4: ${checks}`)




		return !(checks)
}
	function in_check(team){

		let found_king=false
		let king_location = [0,0]
		
		for(let i = 0;!found_king;i++){
			for (let j = 1; j <8; j++){
				let king = board[i].positions[j]

				if (king.charAt(0) == 'K'&&king.charAt(1)==team){
					found_king = true
					king_location[0] = i
					king_location[1] = j

				}
			}
		}
		console.log(`king location: ${king_location}, ${getPiece(king_location[0],king_location[1])}`)
		let opponent
		if (team == 'W') {
			opponent = 'B'
		} else {
			opponent = 'W'
		}
		console.log(`${team} in check`)
		let checks = InCheckByDiagonally(team,king_location)
		//console.log(`check1: ${checks}`)
		checks = checks || InCheckByLaterally(team,king_location)
		//console.log(`check2: ${checks}`)
		checks = checks || InCheckByKing(team,king_location)
		//console.log(`check3: ${checks}`)
		checks = checks || InCheckByKnight(team,king_location)
		//console.log(`check4: ${checks}`)




		return !(checks)
		//add check en passant
		
		
		//return all check functions here

	}

	function getPiece(x,y){
		return board[x].positions[y]
	}
	function getPieceCorrect(x,y){
		return board[y].positions[x]
	}

	function InCheckByKing(team,king_location){
		let x = king_location[0]
		let y = king_location[1]
		//add pawn check in here
		let opponent
		//console.log(`position: ${x},${y} piece: ${getPiece(x-1,y+1)}`)

		if (team == 'W') {
			opponent = 'B'
		} else {
			opponent = 'W'
		}

		if (x < 7){
			if (getPiece(x+1,y).charAt(0) == 'K')
			return true
			if (y<7){
				if (getPiece(x+1,y+1).charAt(0) == 'K')
			return true
			if (getPiece(x+1,y+1).charAt(0) == 'PW' && team == 'B')
			return true
			} else if (y>0){
				if (getPiece(x+1,y-1).charAt(0) == 'PW' && team == 'B')
			return true
			if (getPiece(x+1,y-1).charAt(0) == 'K')
			return true
			}
		}
		if (x > 0){
			if (getPiece(x-1,y).charAt(0) == 'K')
			return true
			if (y<7){
				if (getPiece(x-1,y+1).charAt(0) == 'K')
			return true
			if (getPiece(x-1,y+1).charAt(0) == 'PB' && team == 'W')
			return true
			} else if (y>1){
				if (getPiece(x-1,y-1).charAt(0) == 'K')
			return true
			if (getPiece(x-1,y-1).charAt(0) == 'PB' && team == 'W')
			return true
			}
		}
		if (y<7){
			if (getPiece(x,y+1).charAt(0) == 'K')
			return true
		}
		if (y>1){
			if (getPiece(x,y-1).charAt(0) == 'K')
			return true
		}

		return false
	}

	function InCheckByKnight(team,king_location){
		let opponent;
		if (team == 'W'){
			opponent = 'B'
		} else {
			opponent = 'W'
		}
		let r2_check = king_location[0] >= 0 && king_location[0] <= 5;
		let l2_check = king_location[0] >= 2 && king_location[0] <= 7;
		let d2_check = king_location[1] >= 2 && king_location[1] <= 7;
		let u2_check = king_location[1] >= 0 && king_location[1] <= 5;
		let r1_check = king_location[0] >= 0 && king_location[0] <= 6;
		let l1_check = king_location[0] >= 1 && king_location[0] <= 7;
		let d1_check = king_location[1] >= 1 && king_location[1] <= 7;
		let u1_check = king_location[1] >= 0 && king_location[1] <= 6;

		
		let condition;

		if (r2_check) {

			if (d1_check){

				let piece = getPiece(king_location[0]+2,king_location[1]-1)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'

				if(condition){
					return true
				} 
			} 
			if (u1_check) {
			
				let piece = getPiece(king_location[0]+2,king_location[1]+1)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			}
		} 

		if (l2_check) {

			if (d1_check){
				let piece = getPiece(king_location[0]-2,king_location[1]-1)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			} 
			if (u1_check) {

				let piece = getPiece(king_location[0]-2,king_location[1]+1)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			}
		
		} 

		 if (u2_check) {
			

			if (l1_check){
				

				let piece = getPiece(king_location[0]-1,king_location[1]+2)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			} 
			if (r1_check) {

				let piece = getPiece(king_location[0]+1,king_location[1]+2)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			}
		
		}  
		 if (d2_check) {
			

			if (l1_check){

				let piece = getPiece(king_location[0]-1,king_location[1]-2)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			} 
			 if (r1_check) {
			
				let piece = getPiece(king_location[0]+1,king_location[1]-2)
				condition = piece.charAt(1) == opponent && piece.charAt(0) == 'N'
				if(condition){
					return true
				}
			}
		
		}


		return false


	}

	function InCheckByDiagonally(team,king_location){

		let checks =  InCheckByDiagonal(team,[1,1],king_location)	
		checks = checks || InCheckByDiagonal(team,[1,-1],king_location)	
		checks = checks || InCheckByDiagonal(team,[-1,1],king_location)	
		checks = checks || InCheckByDiagonal(team,[-1,-1],king_location)	

		return checks
	}

	function InCheckByDiagonal(team,direction,king_location){
		let opponent

		if (team == 'W') {
			opponent = 'B'
		} else {
			opponent = 'W'
		}




		let next = [0,0]
		next[0] = king_location[0]
		next[1] = king_location[1]
		next[0] = next[0] + direction[0]
		next[1] = next[1] + direction[1]
		if (next[0] == 8 || next[0] == -1 || next[1] == 8 || next[1] == -1)
		return false

		
		
		let next_space = getPiece(next[0],next[1])
		let next_space_team = next_space.charAt(1)
		let next_space_piece = next_space.charAt(0)
		if (next_space_team == opponent){
			if (next_space_piece == 'Q' || next_space_piece == 'B'){
				console.log(`check from ${next}`)
				return true
			}
			return false
		} else if (next_space_team == ' '){
			return InCheckByDiagonal(team,direction,next)
		} else { 
			return false
		}


	}

	function InCheckByLaterally(team,king_location){
		
		console.log(`check from ${king_location}, ${getPiece(king_location[1],king_location[1])}`)

		let checks = InCheckByDirection(team,1,1,king_location)

		checks = checks || InCheckByDirection(team,-1,1,king_location)

		checks = checks || InCheckByDirection(team,-1,0,king_location)

		checks = checks || InCheckByDirection(team,1,0,king_location)





		return checks
			
	}

	function InCheckByDirection(team,direction,axis,king_location){
		let opponent

		if (team == 'W') {
			opponent = 'B'
		} else {
			opponent = 'W'
		}

		let next = [0,0]
		next[0] = king_location[0]
		next[1] = king_location[1]
		next[axis] = next[axis] + direction
		if (next[axis] == -1 || next[axis] == 8)
		return false
		
		
		let next_space = getPieceCorrect(next[0],next[1])
		let next_space_team = next_space.charAt(1)
		let next_space_piece = next_space.charAt(0)
		if (next_space_team == team) {
			return false
		}else if (next_space_team == opponent){
			if (next_space_piece == 'Q' || next_space_piece == 'R'){
				console.log(`check from ${next}, ${next_space_piece}${next_space_team}`)
				return true
			}
			return false
		} else if (next_space_team == ' '){
			return InCheckByDirection(team,direction,axis,next)
		} else { 
			return false
		}

	}

	function checkDirection(team,direction,axis,location){
		let opponent

		if (team == 'W') {
			opponent = 'B'
		} else {
			opponent = 'W'
		}

		let next = [0,0]
		next[0] = location[0]
		next[1] = location[1]
		next[axis] = next[axis] + direction
		if (next[axis] == -1 || next[axis] == 8)
		return false

		
		
		let next_space_team = board[next[1]].positions[next[0]].charAt(1)
		if (next_space_team == opponent){
			if (next[0] == move_to[0] && next[1] == move_to[1]){
				return true
			}
			return false
		} else if (next_space_team == ' '){
			if (next[0] == move_to[0] && next[1] == move_to[1]){
				return true
			}
			return checkDirection(team,direction,axis,next)
		} else { 
			return false
		}

	}


	function checkKnight(piece,team){
		let opponent;
		if (team == 'W'){
			opponent = 'B'
		} else {
			opponent = 'W'
		}
		let r2_check = move_from[0] >= 0 && move_from[0] <= 5;
		let l2_check = move_from[0] >= 2 && move_from[0] <= 7;
		let d2_check = move_from[1] >= 2 && move_from[1] <= 7;
		let u2_check = move_from[1] >= 0 && move_from[1] <= 5;
		let r1_check = move_from[0] >= 0 && move_from[0] <= 6;
		let l1_check = move_from[0] >= 1 && move_from[0] <= 7;
		let d1_check = move_from[1] >= 1 && move_from[1] <= 7;
		let u1_check = move_from[1] >= 0 && move_from[1] <= 6;

		
		let condition;

		if (r2_check) {
			let r_condition = move_from[0] + 2 == move_to[0] 
			if (d1_check){
				condition =  r_condition && move_from[0] + 2 == move_to[0]
				
				condition =  r_condition && move_from[1] - 1  == move_to[1]
				
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				
				if(condition){
					return true
				} 
			} 
			if (u1_check) {
				condition =  r_condition && move_from[0] + 2 == move_to[0]
				condition =  r_condition && move_from[1] + 1  == move_to[1]
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				if(condition){
					return true
				}
			}
		} 

		if (l2_check) {
			let l_condition = move_from[0] - 2 == move_to[0] 
			if (d1_check){
				condition =  l_condition && move_from[0] - 2 == move_to[0]
				condition =  l_condition && move_from[1] - 1  == move_to[1]
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				if(condition){
					return true
				}
			} 
			if (u1_check) {
				condition =  l_condition && move_from[0] + 2 == move_to[0]
				condition =  l_condition && move_from[1] + 1  == move_to[1]
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				if(condition){
					return true
				}
			}
		
		} 

		 if (u2_check) {
			
			let u_condition = move_from[1] + 2 == move_to[1] 
			if (l1_check){
				
				condition =  u_condition && move_from[1] + 2 == move_to[1]
				
				condition =  u_condition && move_from[0] - 1  == move_to[0]
				
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				
				if(condition){
					return true
				}
			} 
			if (r1_check) {
				condition =  u_condition && move_from[1] + 2 == move_to[1]

				condition =  u_condition && move_from[0] + 1  == move_to[0]

				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				if(condition){
					return true
				}
			}
		
		}  
		 if (d2_check) {
			
			let d_condition = move_from[1] - 2 == move_to[1] 
			if (l1_check){
				condition =  d_condition && move_from[1] - 2 == move_to[1]
				condition =  d_condition && move_from[0] - 1  == move_to[0]
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				if(condition){
					return true
				}
			} 
			 if (r1_check) {
				condition =  d_condition && move_from[1] - 2 == move_to[1]
				condition =  d_condition && move_from[0] + 1  == move_to[0]
				condition = condition && board[move_to[1]].positions[move_to[0]].charAt(1) != team
				if(condition){
					return true
				}
			}
		
		}


		return false


	}

function turnComplete(){
	board = board
			if (turn == 'W'){
				turn = 'B'
			} else {
				turn = 'W'
			}
			last_move[0] = move_to[0]
			last_move[1] = move_to[1]
			
			
			if (turn == 'W'){
				move_no++
			}
}

	function movePiece(row,column){

		if (positions_selected == 1){
			positions_selected = 0;
			move_to = [row-1,column-1];
			if (move_from[0] == move_to[0] && move_from[1] == move_to[1]){
				positions_selected = 1
				return false
			}

			let piece = getPiece(move_from[0],move_from[1])
			let castle_checks
			if (turn == 'W'){
				castle_checks = !w_king_has_moved
				
				if (castle_checks){
					console.log(`c move to ${move_to}`)
					castle_checks = move_to[1] == 0 && (move_to[0]==2 || move_to[0]==6)

					
				}
			} else{
				castle_checks = !b_king_has_moved
				
				if (castle_checks){
					console.log(`c move to ${move_to}`)
					castle_checks = move_to[1] == 7 && (move_to[0]==2 || move_to[0]==6)

					
				}
			}

			if (checkMove()){
			//console.log(`piece: ${board[move_to[1]].positions[move_to[1]]}, row: ${move_to[1]}, column: ${move_to[0]}`)
			//console.log(`piece: ${board[move_from[1]].positions[move_from[1]]}, row: ${move_from[1]}, column: ${move_from[0]}`)

			if (move_to[0] == en_passant[0] && move_to[1] == en_passant[1]){

				let direction
				if (turn == 'W'){
			direction = 1
			} else if (m){

			}
		else {
			direction = -1
		}
			let temp = board[move_to[1]].positions[move_to[0]]
			board[move_to[1]].positions[move_to[0]] = board[move_from[1]].positions[move_from[0]];
			
			board[move_from[1]].positions[move_from[0]] = "  ";

			board[move_to[1]-direction].positions[move_to[0]] = "  ";
			} else{

			let temp = board[move_to[1]].positions[move_to[0]]
			board[move_to[1]].positions[move_to[0]] = board[move_from[1]].positions[move_from[0]];
			board[move_from[1]].positions[move_from[0]] = "  ";
			
		}
			board = board

			if (!(in_check(turn))){
				board[move_from[1]].positions[move_from[0]] = board[move_to[1]].positions[move_to[0]]
				board[move_to[1]].positions[move_to[0]] = temp
				board = board
				console.log("king is in check")
				return false

			}
			board = board
			if (turn == 'W'){
				turn = 'B'
			} else {
				turn = 'W'
			}
			last_move[0] = move_to[0]
			last_move[1] = move_to[1]
			
			
			if (turn == false){
				move_no++
			}
			return true
		} else if(castle_checks){
			console.log(1)
			if (move_to[0] == 2){
				console.log(2)
				let empty_check
				console.log("empty_check")
				empty_check = getPiece(move_from[1],move_from[0]-1) == '  '
				console.log(empty_check)
				empty_check = empty_check && getPiece(move_from[1],move_from[0]-2) == '  '
				console.log(empty_check)
				empty_check = empty_check && getPiece(move_from[1],move_from[0]-3) == '  '
				console.log(empty_check)
				console.log("end empty check")

				if (empty_check){
					console.log(3)
				let check_spaces
				console.log("checking spaces")
				console.log(move_from)
				check_spaces = in_check_location(turn, [move_from[0]-1,move_from[1]])
				console.log(check_spaces)
				check_spaces = check_spaces && in_check_location(turn, [move_from[0]-2,move_from[1]])
				console.log(check_spaces)
				check_spaces = check_spaces && in_check_location(turn, [move_from[0]-3,move_from[1]])
				console.log(check_spaces)
				console.log("end checking spaces")
				if (check_spaces){
					console.log(4)
			let temp = board[move_from[1]].positions[move_from[0]]
			board[move_from[1]].positions[move_from[0]-2] = `K${turn}`
			board[move_from[1]].positions[move_from[0]-1] = `R${turn}`;
			board[move_from[1]].positions[0] = "  ";
			board[move_from[1]].positions[move_from[0]] = "  ";
			turnComplete()
			//need rook has moved check
			}	
			
			} else {
					console.log("spaces not empty")
				}
			} else if (move_to[0] == 6){
				let empty_check

	console.log("empty checks")
	empty_check = getPiece(move_from[1],move_from[0]+1) == '  '
	console.log(empty_check)
	empty_check = empty_check && getPiece(move_from[1],move_from[0]+2) == '  '
	console.log(empty_check)
	console.log("end empty check")


if (empty_check){
	console.log(5)
let check_spaces
check_spaces = in_check_location(turn, [move_from[0]+1,move_from[1]])
console.log(check_spaces)
check_spaces = check_spaces && in_check_location(turn, [move_from[0]+2,move_from[1]])
console.log(check_spaces)
if (check_spaces){
	console.log(6)
let temp = board[move_from[1]].positions[move_from[0]]
board[move_from[1]].positions[move_from[0]+2] = `K${turn}`
board[move_from[1]].positions[move_from[0]+1] = `R${turn}`;
board[move_from[1]].positions[7] = "  ";
board[move_from[1]].positions[move_from[0]] = "  ";
turnComplete()
//need rook has moved check
}	

} else {
	console.log("spaces not empty")
}
			}


		}
		else{
			console.log("invalid move")
			return false
		}
		} else{
		
			positions_selected++;
			move_from = [row-1,column-1];

			return false
		}
	}

	//$: console.log(board)

	function selectCell(row,column){

console.log(row)
console.log(column)

	}
//$: console.log(`move_from: ${move_from}`)
//$: console.log(`move_to: ${move_to}`)
//$: console.log(`en passant: ${en_passant}, move to: ${move_to}`)
</script>

{#each board as row}
<div class="row">
	{#each row.positions as position, i}
	<div class = "cell" on:click="{() => movePiece(i+1,row.row)}">
	{position}

</div>
	{/each}
	</div>
{/each}
<h1>Current turn: {turn} </h1>

<style>
	.row {
		display: flex;
		margin: 4px 0;
	}
	.cell {
		width: 5px;
		height: 5px;
		flex-shrink: 0;
		text-align: center;
		padding: 30px;
		border: 3px solid black;
	}	

</style>