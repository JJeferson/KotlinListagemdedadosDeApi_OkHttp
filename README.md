# KotlinListagemdedadosDeApi_OkHttp
Listagem de dados de API usando ok http com LiveData

# Android Manifest

     <uses-permission android:name="android.permission.INTERNET" />
# Classe que consome a API

    class ConsumoLista()  {


    @Throws(JSONException::class)
    @RequiresApi(Build.VERSION_CODES.N)
    fun Lista_OkHttp(): MutableLiveData<ArrayList<model>> {
        //view: View, adapter: adapter, context: Context
        var LiveData = MutableLiveData<ArrayList<model>>()

        var listaUsuarios = ArrayList<model>()
        listaUsuarios.clear() 




        var policy =
            StrictMode.ThreadPolicy.Builder().permitAll().build()
        StrictMode.setThreadPolicy(policy)

        var client = OkHttpClient().newBuilder()
            .build()


        var request: Request = Request.Builder()
            .url("url da tua web api ")
            .method("GET", null)
            .addHeader("chave_unica", "teste")// header se precisar
            .build()
            // lembre-se que com metodo GET não pode enviar parametro Body
        //--------------------------------------------


        client.newCall(request).enqueue(object : Callback {
            override fun onFailure(call: Call, e: IOException) {}


            @Throws(IOException::class)
            override fun onResponse(call: Call, response: Response) {

                try {
                    var data = response.body()?.string()


                    var jsonArray = JSONArray(data)
                    var contador: Int = 0
                    var tamanhoLista = jsonArray.length()
                    while (contador <= tamanhoLista) {

                        var jsonObject = jsonArray.getJSONObject(contador)

                        contador = contador + 1

                        //pegando do objeto os campos do json que precisa
                        var recebeNome = jsonObject.get("nome").toString()
                        var recebeID_usuario = Integer.parseInt(jsonObject.get("id_usuario").toString())
                        var recebeEmail = jsonObject.get("email").toString()

                        var Model = model(recebeID_usuario, recebeNome, recebeEmail)

                        listaUsuarios.add(Model)
                        LiveData.postValue(listaUsuarios)
                        //então vc guarda os dados do model no arraylist e o arraylist no live data
                    }

                    //-----------------------

                } catch (e: JSONException) {
                    e.printStackTrace()
                }//Final do try

            }//Final do bloco de requisição


        })
     return LiveData
     }

     }
     
 # Adapter
  
      
      class adapter(
      var listaUsuarios: ArrayList<model> = ArrayList(),
      private var viewMain:View,
      private var context: Context,
      UserViewModel: ViewModel): RecyclerView.Adapter<adapter.MyViewHolder>()  {

      var userViewModel = UserViewModel
      var recebeLista = listaUsuarios



       class MyViewHolder(itemView: View): RecyclerView.ViewHolder(itemView) {


       }

       override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {

        return MyViewHolder(LayoutInflater.from(parent.context).inflate(R.layout.layout_lista_recyclerview, parent, false))

      }

      fun addList(lista: ArrayList<model>){

        recebeLista = lista
        notifyDataSetChanged()
      }

      override fun getItemCount(): Int {
        //return userList.size
        return recebeLista.size

      }

       @RequiresApi(Build.VERSION_CODES.N)
       override fun onBindViewHolder(holder: MyViewHolder, position: Int) {

        val currentItem = recebeLista[position]
        var id =  currentItem.id_usuario
        holder.itemView.nomeID.setText("ID: "+id+"  |     Nome do usuario:"+currentItem.nome)
        holder.itemView.emailID.setText("Email: "+currentItem.email)




        holder.itemView.layoutLinhaID.setOnClickListener {view: View ->
            var id = id
            var nome  = currentItem.nome
            var email = currentItem.email


            telasDialog(viewMain,context,userViewModel).criaEditar_ActionDialog(id,nome,email)
        }


        holder.itemView.layoutLinhaID.setOnLongClickListener{
            var id = id
            telasDialog(viewMain,context,userViewModel).criaDELETE_ActionDialog(id)
            //sendo esse metodo onLongClickListner não posso esquecer desse true boolean no final
            true
        }





      }



      }
      
# O observe para montar a lista no teu recyclerview


    @RequiresApi(Build.VERSION_CODES.N)
    private fun setupRecyclerview(){
        teste.setText("")
        val view = window.decorView
        UserViewModel = ViewModelProvider(this).get(ViewModel::class.java)

        adapter = adapter(listaUsuarios,view, this,UserViewModel)
        recyclerViewID.setAdapter(adapter)
        recyclerViewID.layoutManager = LinearLayoutManager (this)
    }

    @RequiresApi(Build.VERSION_CODES.N)
    fun CarregaLista(){
        val view = window.decorView
        UserViewModel = ViewModelProvider(this).get(ViewModel::class.java)



         ConsumoLista().Lista_OkHttp().observe( this,Observer<ArrayList<model>> {it:ArrayList<model> ->
           listaUsuarios.clear()
           listaUsuarios.addAll(it)
           adapter.addList(listaUsuarios)

         })
         
         
# Observação.
Eu tou passando no adapter um viewmodel ali mas ele não é realmente necessario pra isso funcionar.
Apenas ficou ali de destes anteriores pra fazer funcionar.
