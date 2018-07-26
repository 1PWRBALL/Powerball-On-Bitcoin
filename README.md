# Powerball-On-Bitcoin
A Powerball based game that runs on Bitcoin.

Bitcoin Address
BTC: 1PWRBALL6xwyfQnWSSnxVk9bFjq7J42edH


    Powerball on Bitcoin

    The Powerball based game thats available to anyone in the world via Bitcoin
    We use Powerball Drawings to determine our Winners
    Our Algorithm is publicly available and thus 100% verifiable
    All Tickets/Prizes use Bitcoin exchange rate of: 1 Bitcoin = $7000 USD
    WE USE THE SATURDAY POWERBALL DRAWING TO DETERMINE WINNERS, CUTOFF TIME IS 10PM ET


    JACKPOT

    Jackpot Minimum starts at $5,000,000
    If 6% of the Bitcoin Balance is greater then $5,000,000 , then thats the new Jackpot
    Jackpot is split between winners of that drawing


    HOW TO PLAY:

    Tickets = 0.0003 Bitcoin (roughly $2, varies with exchange rate)
    Quick Picks (Limit of 10,000 per Transaction)
    Just send a multiple of 1 tickets price, since each ticket = 0.0003 Bitcoin , 10 tickets = 0.003 Bitcoin
    Or Choose your Powerball and 1 Whiteball
    Choose 1 Powerball (1-26) and 1 White ball (1-69)
    Add these numbers to the ticket after the 3
    example: Base = 0.0003 + 05(PB) + 09(WB) = 0.00030509 Bitcoin
    We will generate the rest of the tickets white balls using those numbers!
    Thats it! whichever way you choose, send to the BTC address above and you are entered to win, any prizes you win will be sent back 
    to the address the ticket was sent from.


    IMPORTANT!!

    USE LOCAL WALLETS ONLY!! DO NOT USE ONLINE WALLETS UNLESS YOU CONTROL THE SENDING ADDRESS
    The reason for this is that in order for you to receive winnings, you must have control over the sending address (input address). 
    Otherwise who ever controls that address will receive the winnings. When sending from a online wallet, depending on how they work, 
    you may not actually have control over the address you sent from. They may send it from an address they just use to store Bitcoin, 
    then keep a DataBase of all the account balances. So when you send, it just subtracts from your account and then uses any Bitcoin 
    addresses it has to fund the transaction. Then if you happen to win, we send your prize back to the address but they would not know 
    who the money belongs too since many users may have used those same addresses. Your money is most likely lost at that point.

    TL:DR,

    It is best to use Local wallets only since you will always have control over any address you send from. So if you win a prize, 
    you will always receive the prize.

    Bitcoin.org Wallets
    Choose either Desktop Wallets, Hardware, Mobile as they are the safest. If you have the space, run a full node like Bitcoin Core or 
    Electrum if you don't.


    HOW IT WORKS

    You give us the PB and WB which we then use to generate the other 4 White balls for a complete Powerball ticket of 1 PB + 5 WB.
    We generate them using your PB + WB + Transaction Data. This way anyone can check what a ticket's 6 numbers are by just looking at 
    the Transaction Data, Then following our algorithm for generating complete Powerball tickets.
    For Quick picks, its the same basic idea as above except instead of only generating 4 white balls, 
    we generate each tickets PB + 5 WB. The seed is generated using only the Transaction Data + modifier for each ball.


    ALGORITHM

    We generate a seed value based on your PB, WB (unless its a quick pick, in which case these are not available to use) , 
    transaction time and transaction id (we remove all Letters and keep the numbers, then shorten it). We then just use the 
    PHP function mt_srand and mt_rand.
    After generating the 4 other White Ball's (or 1PB + 5WB for quick picks), we have a complete Ticket, 
    which we then use to compare against the actual Powerball Drawings. If you win in PowerBall, you win here 
    since we use their numbers to determine winners.
    All our payout's are actually higher then Powerball except for the Jackpots, our Jackpots start at $5,000,000 
    instead of Powerball's $40,000,000, this allows the lower lvl winners (more winners) to win more.


    PHP CODE BELOW, It uses the Codeigniter Framework
    Javascript available by viewing page source

    //Generating Start,
    //This is just the Ticket generation code, all the other code we have is for gathering all Transactions, processing the winning     //tickets (which is really just, does this ticket have any winning numbers, if so what did it win etc..) and finally processing the //Payouts.

    //Values used in both ifs
    //get the blocktime
    $time = floatval($result_array[$i]['time']);

    //Get the numbers in the hash and use as part of seed
    $hash = $result_array[$i]['txid'];
    $formated = str_replace("0", $hash);
    $new_string = preg_replace('/[^0-9]/', '', $formated);

    $hash = $new_string;
    if(strlen($hash) > 9)
    {
        $hash = substr($hash, 0, 9);
    }

    //First make sure the ticket fee minimum was paid, then if 
    //more then 1 tickets worth was paid
    //If 2 or more tickets paid for, randomly generated 
    //tickets = number of tickets paid for, All tickets associated 
    //with same address 

    //get the valueOut to the $lotto_address
    (float)$amnt_paid = floatval($result_array[$i]['valueOut']);
    (int)$total_tickets = floor($amnt_paid / 0.0003);
    (int)$max_tickets = 10000;

    if($total_tickets > $max_tickets)
    {
        $total_tickets = $max_tickets;
    }

    if($amnt_paid >= 0.00030000 && $total_tickets < 2)
    {                      
        $valueout = number_format($result_array[$i]['valueOut'],8);
        //Picks are the 4 last digits, 2 digits per number pick
        $pb   = substr($valueout,(strpos($valueout, ".") + 5 ),2);
        if($pb > 26 || $pb == 0)
        {
            //Generate the PB
            mt_srand($time + (float)$hash + 10000);
            $pb = mt_rand(1,26);
        }

        $wb = array(
            1 => floatval(substr($valueout,(strpos($valueout, ".") + 7 ),2)),
            2 => 0,
            3 => 0,
            4 => 0,
            5 => 0
        );

        if($wb[1] > 69 || $wb[1] == 0)
        {
            mt_srand($time + (float)$hash + 20000);
            $pb = mt_rand(1,69);
        }

        $seed = ($time + (float)$hash + ($wb[1] * $pb));

        //Generate the 4 white balls for a complete ticket, 
        //1 Power Ball + 5 White Balls
        mt_srand($seed + 30000);
        $wb[2] = mt_rand(1,69);

        mt_srand($seed + 40000);
        $wb[3] = mt_rand(1,69);

        mt_srand($seed + 50000);
        $wb[4] = mt_rand(1,69);

        mt_srand($seed + 60000);
        $wb[5] = mt_rand(1,69);

        //Make sure their are no duplicates for whites
        for($d1 = 1; $d1 <= count($wb); $d1++)
        {
            for($d2 = 1; $d2 <= count($wb); $d2++)
            {
                if($wb[$d1] == $wb[$d2] && $d1 != $d2)
                {
                    //change the number
                    for($d3 = 1; $d3 <= 6; $d3++)
                    {
                        //check if it will cause it to go over 69
                        if(($wb[$d2] + $d3) > 69)
                        {
                            if(!in_array(($wb[$d2] - $d3),$wb))
                            {
                                $wb[$d2] = $wb[$d2] - $d3;
                                break;
                            }
                        }
                        else
                        {
                            if(!in_array(($wb[$d2] + $d3),$wb))
                            {
                                $wb[$d2] = $wb[$d2] + $d3;
                                break;
                            }
                        }
                    }
                }
            }
        }

        //add this address and picks to the $tickets array                    
        $data = array(
            'address' => $result_array[$i]['sender_address'],            
            'picks' => array(0 => $pb,
                             1 => $wb[1],
                             2 => $wb[2],
                             3 => $wb[3],
                             4 => $wb[4],
                             5 => $wb[5]
                            )          
            'win_amount' => 0,
            'txs' => $result_array[$i]['txid'],
        );

        array_push($tickets,$data);
    }
    else if($amnt_paid > 0.00030000 && $total_tickets >= 2.0)
    {
        //Loop number of tickets paid for
        for($t = 1; $t <= $total_tickets; $t++)
        {
            //Do the same as above just dont get values after the 3, 
            //instead generate all 6 numbers, 1PB + 5WB
            $pb = 0;

            $wb = array(
                1 => 0,
                2 => 0,
                3 => 0,
                4 => 0,
                5 => 0
            );

            $seed = ($time + (float)$hash + ($t * 100));

            //Generate the power ball
            mt_srand($seed + 10000);
            $pb = mt_rand(1,26);

            //Generate the 5 white balls
            mt_srand($seed + 20000);
            $wb[1] = mt_rand(1,69);

            mt_srand($seed + 30000);
            $wb[2] = mt_rand(1,69);

            mt_srand($seed + 40000);
            $wb[3] = mt_rand(1,69);

            mt_srand($seed + 50000);
            $wb[4] = mt_rand(1,69);

            mt_srand($seed + 60000);
            $wb[5] = mt_rand(1,69);

            //Make sure their are no duplicates for whites
            for($d1 = 1; $d1 <= count($wb); $d1++)
            {
                for($d2 = 1; $d2 <= count($wb); $d2++)
                {
                    if($wb[$d1] == $wb[$d2] && $d1 != $d2)
                    {
                        //change the number
                        for($d3 = 1; $d3 <= 6; $d3++)
                        {
                            //check if it will cause it to go over 69
                            if(($wb[$d2] + $d3) > 69)
                            {
                                if(!in_array(($wb[$d2] - $d3),$wb))
                                {
                                    $wb[$d2] = $wb[$d2] - $d3;
                                    break;
                                }
                            }
                            else
                            {
                                if(!in_array(($wb[$d2] + $d3),$wb))
                                {
                                    $wb[$d2] = $wb[$d2] + $d3;
                                    break;
                                }
                            }
                        }
                    }
                }
            }

            //add this address and picks to the $tickets array                    
            $data = array(
                'address' => $result_array[$i]['sender_address'],            
                'picks' => array(0 => $pb,
                                 1 => $wb[1],
                                 2 => $wb[2],
                                 3 => $wb[3],
                                 4 => $wb[4],
                                 5 => $wb[5]
                                ),          
                'win_amount' => 0,
                'txs' => $result_array[$i]['txid'],
            );

            array_push($tickets,$data);
        }
    }
    else    
    {
        //Ticket fee not paid!
        continue;
    }

    //and then continues processing tickets, checking for winners etc..
    //Generating Finished

