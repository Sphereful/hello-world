override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        initializeWidgets()
        createSpinner()
        initializeRecyclerView()
        totalSpent()
    }
    private fun initializeWidgets() {
        spinner = findViewById(R.id.spinner)
        addButton = findViewById(R.id.add_button)

        spendingInput = findViewById(R.id.spending_et)
        recyclerView = findViewById(R.id.spending_rv)
        totalSpent = findViewById(R.id.total_spent)
        fabAdd = findViewById(R.id.fabAdd)
        fabAdd.setOnClickListener { addToRep() }
        window.setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN)
    }
    private fun createSpinner() {

        ArrayAdapter.createFromResource(
            this,
            R.array.items_array,
            android.R.layout.simple_spinner_dropdown_item
        ).also { adapter ->
            // Specify the layout to use when the list of choices appears
            adapter.setDropDownViewResource(android.R.layout.simple_expandable_list_item_1)
            // Apply the adapter to the spinner
            spinner.adapter = adapter
        }


    }
    private fun initializeRecyclerView() {
        recyclerView.layoutManager = LinearLayoutManager(
            this,
            LinearLayoutManager.VERTICAL, false
        )

        recyclerView.adapter = BudgetRecyclerView(this)
        recyclerView.hasFixedSize()
        deleteSwipe()
    }
    private fun addToRep() {
        val adapter = recyclerView.adapter as BudgetRecyclerView
        adapter.addToBudget(spendingInput, spinner)
        spendingInput.text.clear()
        createNotification()
    }
    private fun deleteSwipe() {
        val swipeHandler = object : SwipeToDelete(this) {
            override fun onSwiped(viewHolder: RecyclerView.ViewHolder, direction: Int) {
                val adapter = recyclerView.adapter as BudgetRecyclerView
                adapter.removeAt(viewHolder.adapterPosition)
                totalSpent()
            }
        }
        val itemTouchHelper = ItemTouchHelper(swipeHandler)
        itemTouchHelper.attachToRecyclerView(recyclerView)
    }
    private val total = expenseDBList.map { it.spendingAmount }.sum()
        //.expenseDBList.
    private val c = NumberFormat.getNumberInstance(Locale.US).format(total).toString()
    private fun totalSpent() {
        totalSpent.text = when (total) {
            in 0..50 -> getString(R.string.woahchilltotal, c)
            in 51..80 -> getString(R.string.stopthismadnesstotal, c)
            else -> {
                getString(R.string.youregoingtojailtotal, c)
            }
        }
    }


    private val notificationId = 1234
    private fun createNotification() {
        val intent = Intent(this, MainActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK or Intent.FLAG_RECEIVER_NO_ABORT
        }
        val pendingIntent: PendingIntent = PendingIntent.getActivity(this, 0, intent, 0)

        val totalString = c
        val notification = NotificationCompat.Builder(this, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_whatshot_black_24dp)
            .setContentText("You've spent $ $totalString this month")
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setContentIntent(pendingIntent)
            .setAutoCancel(false)
            .setOngoing(true)

        with(NotificationManagerCompat.from(this)) {
            notify(notificationId, notification.build())
        }
    }
    ___________________________________________________________________________________________________________________________________________
    
    
    
    
    
    
    
    
    
    
    
     private val expenseRepo = ExpensesRepository(context)
    private val expenseDBList = expenseRepo.findAll()

    /**
     * private fun filteredList(){
    val creationDate = Date()
    val formattedMonth = SimpleDateFormat("yyMMdd")
    val currentMonth = formattedMonth.format(creationDate)
    expenseDBList.filter { it.presentMonth == currentMonth }
    }
     */



    override fun onCreateViewHolder(p0: ViewGroup, p1: Int): BudgetViewHolder {
        val viewHolder = LayoutInflater.from(p0.context)
            .inflate(R.layout.budget_rv_viewholder, p0, false)

        viewHolder.setOnLongClickListener { updateViewHolder(p0.context, p1) } //this creates an alert dialog
        return BudgetViewHolder(viewHolder)
    }

    override fun getItemCount(): Int {
        return expenseDBList.size
    }

    override fun onBindViewHolder(p0: BudgetViewHolder, p1: Int) {

        p0.locationPosition.text = expenseDBList[p1].location
        p0.spendingPosition.text = expenseDBList[p1].spendingAmount.toString()
        p0.datePosition.text = expenseDBList[p1].date

        Log.d(TAG, "onBindViewHolder: ${p0.spendingPosition} , ${p0.locationPosition}")


    }

    fun addToBudget(spendingInput: EditText, spinner: Spinner) {
        if (spendingInput.text.isEmpty()) {
            Toast.makeText(context, "Enter Money Spent, Bitch", Toast.LENGTH_LONG).show()
            return
        } else {
            val number = spendingInput.text.toString().toInt()
            val location = spinner.selectedItem.toString()

            val expenses = BudgetExpense(0, location, number, getCurrentDate())
            expenseRepo.create(expenses)
            expenseDBList.add(expenses)
        }
    }

    private fun getCurrentDate(): String{
        val sdf = SimpleDateFormat("EEE, MMM d")
        return sdf.format(Date())
    }

    private fun updateViewHolder(context: Context, position: Int): Boolean{ //creates a dialog, used in oncreatevh

        val editText = EditText(context)
        AlertDialog.Builder(context)
            .setTitle("Edit")
            .setView(editText)
            .setPositiveButton("Change") { _, _ ->
                val changedText = editText.text.toString()
                val expenseItem = expenseDBList[position]
                expenseItem.location = changedText
                expenseRepo.update(expenseItem)

            }
            .create()
            .show()
        return true
}

    fun removeAt(position: Int) {
        val expenseModel = expenseDBList[position]
        expenseRepo.delete(expenseModel)
       expenseDBList.removeAt(position)

        notifyItemRemoved(position)
        Log.d(TAG, "Removed Model at $position")
    }

    class BudgetViewHolder(itemView: View): RecyclerView.ViewHolder(itemView), View.OnLongClickListener{
        override fun onLongClick(v: View?): Boolean {
            return true
        }

        val locationPosition = itemView.findViewById<TextView>(R.id.location)!!

        val spendingPosition = itemView.findViewById<TextView>(R.id.money)!!

        val datePosition = itemView.findViewById<TextView>(R.id.date)!!

    }
    
    
    
