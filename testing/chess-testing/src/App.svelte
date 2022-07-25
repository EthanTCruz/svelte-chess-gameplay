<script>

	import { board } from './store.js'	

	let turn = true;
	let move_no;
	let positions_selected = 0;
	let move_from;
	let move_to;
	function checkMove(){
		let piece = board[move_from[1]].positions[move_from[0]]
		
		let team = piece.charAt(piece.length - 1);
		piece = piece.charAt(0);
		console.log(team)
		console.log(piece)
		if (piece == 'P'){
			return checkPawn(piece,team)
		}
		return true
	}

	function checkPawn(piece,team){
		let direction;
		let opponent;
		console.log(`move_from: ${move_from}, move_to: ${move_to}`)
		let row_check = move_from[1] >= 1 && move_from[1] <= 6
		let cond1
		let cond2
		let cond3
		let cond4
		let right_check
		let left_check 
		console.log(move_from[0])
		if (team == 'W'){
			direction = 1
			opponent = 'B'
			cond1 = move_from[1]+2*direction == move_to[1] && move_from[1] == 1
		} else {
			direction = -1
			opponent = 'W'
			cond1 = move_from[1]+2*direction == move_to[1] && move_from[1] == 6

		}

		
		right_check = 1 >= move_from[0]+1 && move_from[0]+1 <= 7
		right_check = right_check && board[move_to[1]].positions[move_to[0]].charAt(1) == opponent

		left_check = 2 >= move_from[0]+1 && move_from[0]+1 <= 8
		left_check = left_check && board[move_to[1]].positions[move_to[0]].charAt(1) == opponent
		console.log(1 >= move_from[0]+1 && move_from[0]+1 <= 7)
		console.log(2 >= move_from[0]+1 && move_from[0]+1 <= 8)
		
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

		if (cond1 || cond2 || cond3 || cond4){
			return true
		} else {
			return false
		}

	
	}

	function movePiece(row,column){

		if (positions_selected == 1){
			positions_selected = 0;
			move_to = [row-1,column-1];
			if (checkMove()){
			//console.log(`piece: ${board[move_to[1]].positions[move_to[1]]}, row: ${move_to[1]}, column: ${move_to[0]}`)
			//console.log(`piece: ${board[move_from[1]].positions[move_from[1]]}, row: ${move_from[1]}, column: ${move_from[0]}`)
			board[move_to[1]].positions[move_to[0]] = board[move_from[1]].positions[move_from[0]];
			board = board
			board[move_from[1]].positions[move_from[0]] = "  ";
			board = board
			turn = !turn
			if (turn == false){
				move_no++
			}
		} else{
			console.log("invalid move")
		}
		} else{
		
			positions_selected++;
			move_from = [row-1,column-1];

		}
	}

	//$: console.log(board)

	function selectCell(row,column){

console.log(row)
console.log(column)

	}


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

<style>
	.row {
		display: flex;
		margin: 4px 0;
	}
	.cell {
		width: 40px;
		flex-shrink: 0;
		text-align: center;
		padding: 30px;
		border: 3px solid black;
	}	

</style>